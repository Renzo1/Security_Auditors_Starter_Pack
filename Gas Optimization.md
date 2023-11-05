# Gas Optimization

## Sources

- https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/10
- https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc
- https://github.com/KadenZipfel/gas-optimizations
- [https://www.researchgate.net/publication/340300470\\\_Design\\\_Patterns\\\_for\\\_Gas\\\_Optimization\\\_in\\\_Ethereum](https://www.researchgate.net/publication/340300470%5C_Design%5C_Patterns%5C_for%5C_Gas%5C_Optimization%5C_in%5C_Ethereum)

## Categories

### A. External Transactions

This category includes patterns related to the creation of contracts and the sending of transactions from external addresses, including JavaScript applications, using Web3.js standard library.

| Name: | Proxy |
| --- | --- |
| Problem: | SCs are immutable. If a SC must be changed due to a bug or a needed extension, you must deploy a new contract, and also update all SCs making direct calls to the old SC, thus deploying also new versions of these. This can be very expensive. |
| Solution: | Use Proxy delegate pattern. Proxy patterns are a set of SCs working together to facilitate upgrading of SCs, despite their intrinsic immutability. A Proxy holds the addresses of referred SCs, in its state variables, which can be changed. In this way, only the references to the new SC must be updated. |

&nbsp;

| Name: | Data Contract |
| --- | --- |
| Problem: | When a SC holding a significant amount of data must be updated, also all its data must be copied to the newly deployed SC, consuming a lot of gas. |
| Solution: | Keep the data in a separate SC, accessed by one or more SC, using the data and holding the processing logic. If this logic must be updated, the data remain in the Data Contract. This pattern usually is included also in the implementations of the Proxy pattern. |

&nbsp;

| Name: | Event Log |
| --- | --- |
| Problem: | Often events maintain important information about the system, which must be later used by the external system interacting with the blockchain. Storing this information in the blockchain can be very expensive, if the number of events is high. |
| Solution: | If past events data are needed by the external system, but not by SCs, let the external system directly access the Event Log in the blockchain. Note that this Log is not accessible by SCs, and that if the event happened far in time, the time to retrieve it may be long. |

&nbsp;

| Name: | Checks |
| --- | --- |
| Problem: | checks |
| Solution: | checks |

&nbsp;

### B. Storage

This category includes patterns related to the usage of Storage for storing permanent data.

| Name: | Limit Storage |
| --- | --- |
| Problem: | Storage is by far the most expensive kind of memory, so its usage should be minimized. |
| Solution: | Limit data stored in the blockchain, always use memory for non-permanent data. Also, limit changes in storage: when executing functions, save the intermediate results in memory or stack and update the storage only at the end of all computations. |

&nbsp;

| Name: | Packing Variables |
| --- | --- |
| Problem: | In Ethereum, the minimum unit of memory is a slot of 256 bits. You pay for an integer number of slots even if they are not full. |
| Solution: | Pack the variables. When declaring storage variables, the packable ones, with the same data type, should be declared consecutively. In this way, the packing is done automatically by the Solidity compiler. (Note that this pattern does not work for Memory and Calldata memories, whose variables cannot be packed.) |

&nbsp;

| Name: | Packing Booleans |
| --- | --- |
| Problem: | In Solidity, Boolean variables are stored as uint8 (unsigned integer of 8 bits). However, only 1 bit would be enough to store them. If you need up to 32 Booleans together, you can just follow the Packing Variables pattern. If you need more, you will use more slots than actually needed. |
| Solution: | Pack Booleans in a single uint256 variable. To this purpose, create functions that pack and unpack the Booleans into and from a single variable. The cost of running these functions is cheaper than the cost of extra Storage. |

&nbsp;

| Name: | Gas Costs of Constant and Immutable Variables |
| --- | --- |
| Problem: | Gas costs for constant and immutable variables are much lower compared to regular state variables. |
| Solution: | If a storage variable must be assigned at deployment and doesn't need to be reassigned, it should be marked as immutable.  <br>Better yet, if it doesn't need to be assigned at deployment then it can be set as a constant. |
| 1.  | **Constant Variables:** The expression assigned to constant variables is copied to all the places where it's accessed and re-evaluated each time, allowing for local optimizations. |
| 2.  | **Immutable Variables:** Immutable variables are evaluated once at construction time, and their value is copied to all code locations where they are accessed. Note that 32 bytes are reserved for these values, even if they would fit in fewer bytes. Therefore, constant values can sometimes be cheaper than immutable values. |

&nbsp;

| Name: | User Allowance Default Value Optimization |
| --- | --- |
| Problem: | Defaulting the user's allowance to 2^256 - 1 (0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff) is common in projects like Uniswap, but it incurs higher gas costs due to the hex representation of this value. |
| Solution: |     |
| 1.  | Consider using 0x8000000000000000000000000000000000000000000000000000000000000000 (2^255) as an alternative representation for "infinity" in allowance values. |
| 2.  | The gas cost of 0x8000000000000000000000000000000000000000000000000000000000000000 is 45,888 gas, compared to 47,872 gas for 2^256 - 1. This optimization can save 1,984 gas (4.1%). |
| Concern (further research) | Could this have some security implication. e.g. the overflow guard of maxUint256 is lost when we bound with a lower value |

&nbsp;

| Name: | Proper Data Types |
| --- | --- |
| Problem: | In Solidity, some data types are more expensive than others. It’s valuable to know the most efficient type that can be used. Here are a few rules about data types. |
| Solution: | \- Type uint or bytes32 should be used in place of type string whenever possible.  <br>\- Type uint256 takes less gas to store than other uint types (\[see why\](https://ethereum.stackexchange.com/questions/3067/why-does-uint8-cost-more-gas-than-uint256)).  <br>\- Type bytes should be used over byte\[\].  <br>\- If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32. |


&nbsp;

| Name: | [Struct Bit Packing for Gas Efficiency](https://github.com/KadenZipfel/gas-optimizations/blob/main/gas-saving-patterns/struct-bit-packing.md#struct-bit-packing) |
| --- | --- |
| Problem: | Solidity's storage layout consists of 2\*\*256 32-byte slots, and each read or write operation comes with a fixed cost per slot. To minimize gas costs, it's crucial to use as few slots as possible. |
| Solution: | Implement struct bit packing to optimize gas consumption. By packing variables efficiently, you can reduce the number of slots used in storage.



&nbsp;

| Name: | Checks |
| --- | --- |
| Problem: | checks |
| Solution: | checks |


&nbsp;

### C. Saving Space

This category includes patterns related to saving space both in Memory and Storage.

| Name: | Uint\* vs Uint256 |
| --- | --- |
| Problem: | The EVM run on 256 bits at a time, thus using an uint\* (unsigned integers smaller than 256 bits), it will first be converted to uint256 and it costs extra gas. |
| Solution: | Use unsigned integers smaller or equal than 128 bits when packing more variables in one slot (see Variables Packing pattern). If not, it is better to use uint256 variables. |

&nbsp;

| Name: | Mapping vs Array |
| --- | --- |
| Problem: | Solidity provides only two data types to represents list of data: arrays and maps. Mappings are cheaper, while arrays are packable and iterable. |
| Solution: | In order to save gas, it is recommended to use mappings to manage lists of data, unless there is a need to iterate or it is possible to pack data types. This is useful both for Storage and Memory. You can manage an ordered list with a mapping using an integer index as a key. |

&nbsp;

| Name: | Fixed Size |
| --- | --- |
| Problem: | In Solidity, any fixed size variable is cheaper than variable size. |
| Solution: | Whenever it is possible to set an upper bound on the size of an array, use a fixed size array instead of a dynamic one. e.g. If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. |

&nbsp;

| Name: | Default Value |
| --- | --- |
| Problem: | It is good software engineering practice to initialize all variables when they are created. However, this costs gas in Ethereum. |
| Solution: | In Solidity, all variables are set to zeroes by default. So, do not explicitly initialize a variable with its default value if it is zero. |

&nbsp;

| Name: | Minimize on-chain data |
| --- | --- |
| Problem: | The gas costs of Storage are very high, and much higher than the cost of Memory. |
| Solution: | Minimize on-chain data. The less data you put on-chain in Storage variables, the less your gas costs. Store on-chain only critical data for the SC and keep all possible data off-chain. |

&nbsp;

| Name: | Explicitly mark external function |
| --- | --- |
| Problem: | The input parameters of public functions are copied to memory automatically, and this costs gas. |
| Solution: | The input parameters of external functions are read right from Calldata memory. Therefore, explicitly mark as external functions called only externally. |

&nbsp;  
Here's your fifth gas optimization note formatted according to the provided template:

| Name: | Use calldata Instead of Memory for Function Parameters |
| --- | --- |
| Problem: | In some cases, using function arguments in memory instead of calldata can result in inefficiencies. |
| Solution: | Consider using calldata for function parameters, especially when the argument is only read within the function. |

&nbsp;

| Name: | Checks |
| --- | --- |
| Problem: | checks |
| Solution: | checks |

&nbsp;

### D. Operations

This category includes patterns related to the gas used for the operations performed within SC functions.

| Name: | Limit External Calls |
| --- | --- |
| Problem: | Every call to an external SC is rather expensive, and even potentially unsafe. |
| Solution: | Limit external calls. In Solidity, differently from other programming languages, it is better to call a single, multi-purpose function with many parameters and get back the requested results, rather than making different calls for each data. |

&nbsp;

| Name: | Internal Function Calls |
| --- | --- |
| Problem: | Calling public functions is more expensive than calling internal functions, because in the former case all the parameters are copied into Memory. |
| Solution: | Whenever possible, prefer internal function calls, where the parameters are passed as references. |

&nbsp;

| Name: | Fewer functions |
| --- | --- |
| Problem: | Implementing a function in an Ethereum SC costs gas. |
| Solution: | In general, keep in mind that implementing a SC with many small functions is expensive. However, having too big functions complicates the testing and potentially compromises the security. So, try to have fewer functions, but not too few, balancing the function number with their complexity. |

&nbsp;

| Name: | Use Libraries |
| --- | --- |
| Problem: | If a SC tends to perform all its tasks by its own code, it will grow and be very expensive. |
| Solution: | Use libraries. The bytecode of external libraries is not part of your SC, thus saving gas. However, calling them is costly and has security issues. Use libraries in a balanced way, for complex tasks. |

&nbsp;

| Name: | Short Circuit |
| --- | --- |
| Problem: | Every single operation costs gas. |
| Solution: | When using the logical operators, order the expressions to reduce the probability of evaluating the second expression. Remember that in the logical disjunction (OR, \|), if the first expression resolves to true, the second one will not be executed; or that in the logical disjunction (AND, &&), if the first expression is evaluated as false, the next one will not be evaluated. |

&nbsp;

| Name: | Short Constant Strings |
| --- | --- |
| Problem: | Storing strings is costly. |
| Solution: | Keep constant strings short. Be sure that constant strings fit 32 bytes. For example, it is possible to clarify an error using a string; these messages, however, are included in the bytecode, so they must be kept short to avoid wasting memory. |

&nbsp;

| Name: | Limit Modifiers |
| --- | --- |
| Problem: | The code of modifiers is inlined inside the modified function, thus adding up size and costing gas. |
| Solution: | Limit the modifiers. Internal functions are not inlined, but called as separate functions. They are slightly more expensive at run time, but save a lot of redundant bytecode in deployment, if used more than once. |

&nbsp;

| Name: | Avoid redundant operations |
| --- | --- |
| Problem: | Every single operation costs gas. |
| Solution: | Avoid redundant operations. For instance, avoid double checks; the use of SafeMath library prevents underflow and overflow, so there is no need to check for them. |

&nbsp;

| Name: | Single Line Swap |
| --- | --- |
| Problem: | Each assignment and defining variables costs gas. |
| Solution: | Solidity allows to swap the values of two variables in one instruction. So, instead of the classical swap using an auxiliary variable, use: (a, b) = (b, a) |

&nbsp;

| Name: | Write Values |
| --- | --- |
| Problem: | Every single operation costs gas. |
| Solution: | Write values instead of computing them. If you already know the value of some data at compile time, write directly these values. Do not use Solidity functions to derive the value of the data during their initialization. Doing so, might lead to a less clear code, but it saves gas. |

&nbsp;

| Name: | [Constructor Payable Gas Optimization](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/5?u=kris_renzo) |
| --- | --- |
| Problem: | During EVM bytecode creation, certain opcodes can be eliminated, saving gas. |
| Solution: | Declare the constructor as payable to remove the following opcodes from the EVM bytecode |

&nbsp;

| Name: | [Upgrade to Compiler Version 0.8.4 and Above](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6#upgrade-to-at-least-084-1) |
| --- | --- |
| Problem: | Using older compiler versions may result in missed gas optimizations and safety checks. |
| Solution: |     |
| 1.  | Upgrade your Solidity compiler to at least version 0.8.4 or higher to benefit from gas optimizations and enhanced safety checks. |
| 2.  | Advantages of using versions 0.8.\* over versions below 0.8.0 include: |

&nbsp;  
Here's your sixth gas optimization note formatted using the provided template:

| Name: | [Use Unchecked Increment in for Loop Postcondition](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6#the-increment-in-for-loop-postcondition-can-be-made-unchecked-6) |
| --- | --- |
| Problem: | In Solidity, for loop postconditions include checked arithmetic, even when it's unnecessary, resulting in gas inefficiencies. |
| Solution: | Consider using unchecked increments in the postcondition of for loops to save gas, especially when using the default Solidity checked arithmetic. |

&nbsp;

| Name: | Use Custom Errors Instead of Revert Strings |
| --- | --- |
| Problem: | Prior to Solidity 0.8.4, using revert strings incurred unnecessary gas costs for deployment and runtime when the revert condition was met. |
| Solution: | Starting from Solidity 0.8.4, custom errors were introduced, offering a more gas-efficient alternative to revert strings for both deployment and runtime cost. Consider replacing revert strings with custom errors for gas savings. |

&nbsp;

| Name: | Use `!= 0` Instead of `> 0` for Unsigned Integer Comparison |
| --- | --- |
| Problem: | When working with unsigned integer types, using `!= 0` in comparisons is more gas-efficient than using `> 0`. |
| Solution: | Opt for `!= 0` comparisons when dealing with unsigned integer types, as they are more cost-effective in terms of gas consumption. |
| Note:(Validate this) | With using at least 0.8.6 there is no difference in gas usage with!= 0 or > 0 |

&nbsp;

| Name: | Optimal Comparison Operator Selection |
| --- | --- |
| Problem: | Gas costs can be reduced by carefully selecting the optimal comparison operator when dealing with uints. |
| Solution: | For example, it is cheaper to use strict < or > operators over <= or >= operators. This is due to the additional EQ opcode which must be performed. In the case of a conditional statement, it is further optimal to use != when possible.|


&nbsp;

| Name: | Use Shift Right/Left Instead of Division/Multiplication If Possible |
| --- | --- |
| Problem: | Solidity's DIV/MUL opcodes use 5 gas, whereas the SHR/SHL opcodes only use 3 gas. Additionally, the division operation includes division-by-0 prevention, which can be bypassed using shifting. Overflow checks are also not performed for shift operations; instead, the result is truncated. |
| Solution: | Consider using shift right (SHR) and shift left (SHL) operations instead of division and multiplication when possible to save on gas costs. |
|     | \`\`\`solidity  <br>// Use shift right (>>) instead of division (/)  <br>uint256 result = valueToDivide >> 1; // Equivalent to valueToDivide / 2  <br>  <br>// Use shift left (<<) instead of multiplication (\*)  <br>uint256 result = valueToMultiply << 1; // Equivalent to valueToMultiply \* 2  <br>\`\`\` |
| Concerns | Could break any security protections or checks that comes with regular arithmetic symbols. Also the gas difference save may not be significant enough for the risk. Best use situations like an array iteration |

&nbsp;

| Name: | Utilize Fallback Function and Sig-less Functions |
| --- | --- |
| Problem: | Gas efficiency can be improved by using the fallback function (in Solidity) or sig-less functions (in Yul) as they do not require a function signature for invocation. |
| Solution: | Explore the use of the fallback function in Solidity or sig-less functions in Yul where applicable to reduce gas costs. |

&nbsp;

| Name: | Minimize Negative Values in Calldata |
| --- | --- |
| Problem: | Negative values in calldata are more expensive due to the additional 0xf bytes that are prepended to them. This leads to higher gas costs. |
| Solution: | Aim to minimize the use of negative values in calldata to reduce gas costs, as they are represented with additional 0xf bytes. |

&nbsp;

| Name: | Consider Using Payable Functions When Appropriate |
| --- | --- |
| Problem: | Functions that are not explicitly marked as payable incur a 21 gas cost for an initial check that msg.value == 0. In some cases, this check may not provide any benefit, making it more gas-efficient to mark the function as payable. |
| Solution: | Evaluate the necessity (security trade-off) of marking a function as payable based on your specific use case.|

&nbsp;

### E. Miscellaneous

This category includes patterns that cannot be included in the previous ones.

| Name: | Freeing storage |
| --- | --- |
| Problem: | Sometimes, Storage variables are not longer used. Is there a way to take advantage of this? |
| Solution: | To help keeping the size of the blockchain smaller, you get a gas refund every time you free the Storage. Therefore, it is convenient to delete the variables on the Storage, using the keyword delete, as soon as they are no longer necessary. |

&nbsp;

| Name: | Optimizer |
| --- | --- |
| Problem: | Optimizing Solidity code to save gas in exhaustive way is difficult. |
| Solution: | Always turn on the Solidity Optimizer. It is an option of all Solidity compilers, which performs all the optimizations that can be made by the compiler. However, it does not substitute the usage of the presented patterns, most of which need information that is not available to the compiler. |

&nbsp;

| Name: | Use short reason strings |
| --- | --- |
| Problem: | You can (and should) attach error reason strings along with `require` statements to make it easier to understand why a contract call reverted. These strings, however, take space in the deployed bytecode. |
| Solution: | Every reason string takes at least 32 bytes so make sure your string fits in 32 bytes or it will become more expensive. |

&nbsp;

| Name: | Make proper use of the optimizer |
| --- | --- |
| Problem: | Apart from allowing you to turn optimizer on and off, solc allows you to customize optimizer runs. `runs` is not how many times the optimizer will run but how many times you expect to call functions in that smart contract. |
| Solution: | If the smart contract is only of one-time use as a smart contract for vesting or locking of tokens, you can set the `runs` value to `1` so that the compiler will produce the smallest possible bytecode but it may cost slightly more gas to call the function(s). If you are deploying a contract that will be used a lot (like an ERC20 token), you should set the `runs` to a high number like `1337` so that initial bytecode will be slightly larger but calls made to that contract will be cheaper. Commonly used functions like transfer will be cheaper. |

&nbsp;

| Name: | Checks |
| --- | --- |
| Problem: | checks |
| Solution: | checks |

&nbsp;

### F. Gas Costly Patterns

#### Comparison with Unilateral Outcome in a Loop

When a comparison is executed in each iteration of a loop but the result is the same each time, it should be removed from the loop.

For example, the following:

```
function unilateralOutcome(uint x) public returns(uint) {
  uint sum = 0;
  for(uint i = 0; i <= 100; i++) {
    if(x > 1) {
      sum += 1;
    }
  }
  return sum;
}
```

Should be re-written as:

```
function noUnilateralOutcome(uint x) public returns(uint) {
  uint sum = 0;
  bool increment = x > 1;
  for(uint i = 0; i <= 100; i++) {
    if(increment) {
      sum += 1;
    }
  }
  return sum;
}
```

&nbsp;

#### Constant Outcome of a Loop

If the outcome of a loop is a constant that can be inferred during compilation, it should not be used.

```
function constantOutcome() public pure returns(uint) {
  uint num = 0;
  for(uint i = 0; i < 100; i++) {
    num += 1;
  }
  return num;
}
```

&nbsp;

#### Dead Code

Dead code is code that will never run because it’s evaluation is predicated on a condition that will always return false.

```
function deadCode(uint x) public pure {
  if(x < 1) {
    if(x > 2) {
      // this will never run
      return x;
    }
  }
}
```

&nbsp;

#### Loop Fusion

Occasionally in smart contracts, you may find that there are two loops with the same parameters. In the case that the loop parameters are the same, they should be combined if possible.

```
 function loopFusion(uint x, uint y) public pure returns(uint) {
  for(uint i = 0; i < 100; i++) {
    x += 1;
  }
  for(uint i = 0; i < 100; i++) {
    y += 1;
  }
  return x + y;
}
```

Should be re-written as:

```
 function lessExpensiveLoopFusion(uint x, uint y) public pure returns(uint) {
  for(uint i = 0; i < 100; i++) {
    x += 1;
    y += 1;
  }
  return x + y;
}
```

&nbsp;

#### Opaque Predicate

The outcome of some conditions can be known without being executed and thus do not need to be evaluated.

```
function opaquePredicate(uint x) public pure {
  if(x > 1) {
    // redundant
    if(x > 0) {
      return x;
    }
  }
}
```

&nbsp;

#### Repeated Computations in a Loop

If an expression in a loop produces the same outcome in each iteration, it can be moved out of the loop. This is especially important when the variables(s) used in the expression are stored in storage.

```
uint a = 4;
uint b = 5;
function repeatedComputations(uint x) public returns(uint) {
  uint sum = 0;
  for(uint i = 0; i <= x; i++) {
    sum = a + b;
  }
  return sum;
}
```

&nbsp;

#### Unnecessary Libraries

Libraries are often only imported for a small number of uses, meaning that they can contain a significant amount of code that is redundant to your contract. If you can safely and effectively implement the functionality imported from a library within your contract, it is optimal to do so.

```
import './SafeMath.sol' as SafeMath;

// redundant library
contract SafeAddition {
  function safeAdd(uint a, uint b) public pure returns(uint) {
    return SafeMath.add(a, b);
  }
}
```

```
// w/o library
contract SafeAddition {
  function safeAdd(uint a, uint b) public pure returns(uint) {
    uint c = a + b;
    require(c >= a, "Addition overflow");
    return c;
  }
}
```

&nbsp;

#### Function Ordering
When a function is called, the EVM compares the method ID of the transaction to the method ID's present in the contract, with each comparison costing an additional 22 gas. These comparisons are performed in order from least to greatest byte value of method ID's (alphanumerically based on the method ID). This is done using binary search.

For this reason:
**1. Have method name with IDs that have more 0s.**
As with calldata, having a method name whose function selector has more 0s helps.
```
// method ID: 0x6a761202
execTransaction()

// method ID: 0x00000081
execTransaction32785586()
```
This saves 192 bytes of data.

**2. Reorder methods based on call frequency**
there is some value in renaming functions to be ordered based on the frequency you expect them to be used, saving average overall gas cost of interacting with your contract.

```
// method ID: 0x1249c58b
mint()

// method ID: 0x0000b696
mint_Aci()
```

Useful tool: https://emn178.github.io/solidity-optimize-name/

&nbsp;

#### Using Mulmod Over Mul & Div

If you have to check whether a value obtained by dividing it by a number is exact(i.e. significant digits have not truncated during division), use `mulmod` instead of mul & div as it costs 8 gas. In contrast, `mul` and `div` cost 5 gas each.

- Gas inefficient method
```
    contract UsingMul&Div {
        error InExactDivision();

        function exactDivide(
            uint256 numerator,
            uint256 denominator,
            uint256 value
        ) public pure returns (uint256 newValue) {
            newValue = (value * numerator)/denominator;
            if(value != (newValue * denominator)/ numerator) {
                revert InExactDivision();
            }
        }
    }

```

- Gas efficient method

```
    contract UsingMulMod {
        error InExactDivision();

        function exactDivide(
            uint256 numerator,
            uint256 denominator,
            uint256 value
        ) public pure returns (uint256 newValue) {
            newValue = (value * numerator)/denominator;
            if(mulmod(value, numerator, denominator) != 0) {
                revert InExactDivision();
            }
        }
    }

```

- Gas efficient method(With Assembly)

```
    contract UsingMulModWithAssembly {
        error InExactDivision();

        function exactDivide(
            uint256 numerator,
            uint256 denominator,
            uint256 value
        ) public pure returns (uint256 newValue) {
            bool inexact;
            assembly {
                newValue := div(mul(value, numerator, denominator));
                inexact := iszero(iszero(mulmod(value, numerator, denominator)));
            }
            if(inexact) {
                revert InExactDivision();
            }
        }
    }

```

#### Optimal Increment and Decrement Operators

Increment and decrement operators are used to either increment or decrement their operand by one. These incrementers/decrementers can either be presented as e.g. `i++` or `++i`, with the only difference being the order of operations being executed. `i++` will return `i`, then increment the value of `i`, whereas `++i` will increment before returning the value.

As a result of the variance in order of operations, `++i` will save two opcodes from being performed. Consider the following example:

```
pragma solidity ^0.8.0;

contract IPlusPlus {
    // 1996 gas
    function loop() external pure {
        for (uint256 i; i < 10; i++) {
            // do something
        }
    }
}

contract PlusPlusI {
    // 1946 gas
    function loop() external pure {
        for (uint256 i; i < 10; ++i) {
            // do something
        }
    }
}
```

Using `++i` above shaves off 5 gas per loop. This is because `i++` must handle the value of `i` both before and after the increment, in this case by executing a `DUPN` opcode (3 gas) and later cleaning up the stack by performing a `POP` (2 gas).
