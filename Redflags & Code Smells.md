# Red-flag Alerts - Terms & Set-ups to Immediately Search in Every Contract

`To do: Scroll through Security pitfalls and update this list`

## Terms to Immediate Search in Every Contract

- *tx.origin*
- *ecrecover*
- *selfdestruct*
- *delegatecall*
- *callcode*
- *staticcall*
- *send*
- *transfer*
- *block.timestamp*
- *block.number*
- *this.balance*
- *delete*
- *external*
- *public*
- *gasleft*
- *blockhash*
- *extcodesize*
- *abi.encodePacked*
- *abi.encode*
- *constant*
- *true (boolean constants)*
- *false (boolean constants)*
- *for (Calls inside a loop; Modifying an array of unknown size)*
- *while (Calls inside a loop)*
- *block.blockhash() for blockhash() (Deprecated keywords)*
- *msg.gas for gasleft() (Deprecated keywords)*
- *throw for revert() (Deprecated keywords)*
- *sha3() for keccak256() (Deprecated keywords)*
- *callcode() for delegatecall() (Deprecated keywords)*
- *suicide() for selfdestruct() (Deprecated keywords)*
- *constant for view (Deprecated keywords)*
- *var for actual type name (Deprecated keywords)*
- *abi.encodePacked() (Hash collisions)*
- *assembly*
- *ABIEncoderV2 (solidity versions <0.7.0)*
- *abi (solidity versions <0.7.0)*
- *v0.7.*
- *v0.6*
- *v0.5.*
- *v0.4.*
- *initialize*
- *initializer*
- *Initializable*
- *ERC20 (ERC20 decimals returns a uint8)*
- *ERC777*
- *ERC1400*
- *Owner*
- *mapping*
- *permit*
- *some\_collision*
- *EIP-2612*
- *DOMAIN\_SEPARATOR*
- *event*
- *using*
- *this*
- */ (division sign)* : divide by zero case or precision loss (round down to zero -- when numerator is significantly smaller the denominator)
- *create2*

&nbsp;

## *Code Smells & Attack Vectors to look out for*

- Analyze non-overridden functions of inherited

- Contract guardrails (protection mechanisms) not being set during construction (in the constructor).
    
- Not using(inheriting) an upgradable variant of an Openzeppelin contract in an upgrable smart contract
    
- Catch block actually catch return value -- catch (bytes memory returnData) {} (ref: https://youtu.be/DRZogmD647U?t=5380)
    
- No Input Validation in function
    
- Named import syntax not used
    
- Solidity version less than 0.8.0
    
- Calling a public or external function from within another function
    
- Exposing initialization functions: wrongly naming a function intended to be a constructor, the constructor code ends up in the runtime byte code and can be called by anyone to re-initialize the contract.
    
- Modifying an array of unknown size, that increases in size over time, can lead to such a Denial of Service condition.
    
- Look out for Symmetric & Asymmetry in a function block (mint/burn, buy/sell, repaying calculations in multiple places).
    
- Reused base constructors
    
- Examine every external calls for re-entrancies
    
- Check if external calls can DoS by reverting or spending a lot of gas
    
- Check if contracts receiving ether has a withdraw or fallback function
    
- Check if parallel data structures always in sync
    
- Check for Arrays that are too long to delete
    
- Check for jagged arrays - two arrays passed in that are not the same length
    
- 63 out of 64 rule
    
- Flash-Loanable ERC-721 tokens
    
- Gas Tokens to extract value from Gas Griefing
    
- Struct being replaced by an empty uninitialized version with zeroed out critical values
    
- Multicall functions with a payable modifier
    
- Using msg.value in a loop
    
- Decoding arbitrary bytes that come from an untrusted address
    
- Fee-on-transfer
    
- Rebasing tokens
    
- Chain Incompactibility
    
- In Inline Assemby MSTORE does not update the free memory pointer
    
- Using Transfer or Send will often preclude multi-sig wallets
    
- Precision loss or divide by zero reverts with division symbols
    
- Handling token units wrong
    
- Assuming every address is able to accept ether or ERC20 tokens
    
- Loading in the return value of .call into memory
    
- Using for loops to push rather than pull
    
- Delete on structs doesn't delete contained mapping or dynamic lists
    
- Immutable values are not maintained on upgrade since these values are a part of the contract code
    
- Subtractions that underflow and revert
    
- Down-casting can still overflow
    
- Parallel data structures
    
- Typos
    
- Rounding errors
    
- A \`Multicall\` function that works with \`address(this).delegatecall()\` and using \`msg.value\`. In a "delegatecall" the \`msg.value\` is reused, hence it can be used for a double-spend exploit
    

&nbsp;

&nbsp;

## *Notes*

- Every Ether transfer implies potential code execution.
    - The receiving address can implement a fallback function that can throw an error. Thus, we should never trust that a `send` call will execute without error. A solution: our contracts should favor pull over push for payments.
- External calls can fail accidentally or deliberately.
    - To minimize external call failures, isolate each call in its transaction, initiated by the recipient, especially for payments. This way, users withdraw funds when needed, reducing gas limit problems. Avoid combining multiple send() calls in one transaction.
- Non–token-related functions increase the likelihood of an issue in the contract.
- The token should be a simple contract
- How can a malicious token affect this contract?
- System specification:
    - Comprehensive system specification is crucial as it details how system components behave to meet design requirements. It enables the evaluation of correctness during system implementation, ensuring alignment with design goals.
- State changing calls that aren't verified
- Implicit requirements and assumptions should be flagged as dangerous.
- Incorrect usage of using-for statement
- What if I call with an empty list? With a really big number? With a really small number (1 wei)? With identical addresses?
- Stepwise jumps: [Frontrunning](https://youtu.be/DRZogmD647U?t=15261)

Token Attack
