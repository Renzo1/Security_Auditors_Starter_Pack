# Red-flag, Code Smells, and Heuristics

`To do: Scroll through Security pitfalls, attacks, research papers, other works and update this list`

## Redflag

tx.origin  
ecrecover  
selfdestruct  
delegatecall  
callcode  
staticcall  
send  
transfer  
block.timestamp  
block.number  
this.balance  
delete  
external  
public  
gasleft  
blockhash  
extcodesize  
abi.encodePacked  
abi.encode  
constant  
true (boolean constants)  
false (boolean constants)  
for (Calls inside a loop; Modifying an array of unknown size)  
while (Calls inside a loop)  
block.blockhash() for blockhash() (Deprecated keywords)  
msg.gas for gasleft() (Deprecated keywords)  
throw for revert() (Deprecated keywords)  
sha3() for keccak256() (Deprecated keywords)  
callcode() for delegatecall() (Deprecated keywords)  
suicide() for selfdestruct() (Deprecated keywords)  
constant for view (Deprecated keywords)  
var for actual type name (Deprecated keywords)  
abi.encodePacked() (Hash collisions)  
assembly  
ABIEncoderV2 (solidity versions <0.7.0)  
abi (solidity versions <0.7.0)  
v0.7.  
v0.6  
v0.5.  
v0.4.  
initialize  
initializer  
Initializable  
ERC20 (ERC20 decimals returns a uint8) and  (instead of ERC20)  
ERC777  
ERC1400  
Owner  
mapping  
permit  
some_collision  
EIP-2612  
DOMAIN_SEPARATOR  
event  
using  
this  
/ (division sign) : divide by zero case or precision loss (round down to zero -- when numerator is significantly smaller the denominator)  
create2  
owner == address(0)  
TODO  
to do  
to-do  
address(msg.sender)  
transferFrom (instead of safeTransferFrom)  
int  
modifier  
\=+  
ERC1400  
ERC621  
ERC884  
ERC721  
using  
msg.value  
uint256(  
uint128(  
uint96(  
uint64(  
uint32(  
uint16(  
uint8(  
call

&nbsp;

### NEW

!=  
users.length  
addresses.length  
transferFrom(  
assert(  
balanceOf(address(this))

&nbsp;

&nbsp;

## Covered by Testing and Formal Verification

**User Input and External Interaction:**

- Can a user pass an arbitrary input?
- What if I call with an empty list? With a really big number? With a really small number (1 wei)? With identical addresses?

&nbsp;

**Contract Design, Inheritance, and Execution:**

- Does this function break if I call it more than once?
- Do many small operations result in the same effect as one large operation?
- Does "total" (stored in its storage location) equal the sum of all individual user values (stored in a mapping) at all times? What about at snapshot time?
- When a snapshot is taken, can the order of function calls lead to an inconsistency between the "total" storage location and the sum of individual user values?
- Anything that has a public or external attribute, dependencies on other contracts, or state changes — anything that triggers behavior changes based on inputs are prone to malicious entry points.
- Reused base constructors
- Over-inheritance
    - Check if inherited contracts are overwriting each other
    - Check if a contract is inherited more than once
- Named import syntax not used
- Analyze non-overridden functions of inherited contracts (non-utilized functions should be overridden and disabled)
- Division sign (/)
    - Divide by zero revert
    - Precision error
    - Truncation
        - Are they expecting truncation to occur?
        - Are they handling truncation the right way?
        - What can happen if I provide really small values? Can that get rounded off and after the system?
- [Calling a function X times with value Y == Calling it once with value XY](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#calling-a-function-x-times-with-value-y--calling-it-once-with-value-xy)

&nbsp;

**Security Considerations:**

- Handling [decimals](https://youtu.be/DRZogmD647U?t=27234)
- Tokens with multiple entry points for balance updates can break internal bookkeeping based on the address (e.g., *balances\[token_address\]\[msg.sender\]* might not reflect the actual balance).
- Ensure that the right set of function modifiers are used (in the correct order) for the specific functions so that the expected access control or validation is correctly enforced.
- Ensure that the appropriate return value(s) are returned from functions and checked without ignoring at function call sites.
- Are there any assumptions concerning:
    - Function invocation timeliness
    - Function invocation repetitiveness
    - Function invocation order
    - Function invocation arguments
- Contract guardrails (protection mechanisms) not being set during construction (in the constructor)
- No Input Validation in function
- Modifying storage array by value
- Look out for Symmetric & Asymmetry in a function block (mint/burn, buy/sell, repaying calculations in multiple places).
- Type casting uint to int and vice versa (use a safeCast Library)
- Dependence on Solidity's arithmetic operator precedence rules
- Operating on more than one address
    - Dividing before Multiplying
- [Rounding errors that can be amplified](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#rounding-errors-that-can-be-amplified)
- [Off-by-one errors](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#off-by-one-errors): Is <= correct in this context or should < be used? Should a variable be set to the length of a list or the length - 1? Should an iteration start at 1 or 0?
- [Duplicates in lists](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#duplicates-in-lists): Thinking about what happens when there are duplicates (and if it is possible) can be very helpful

**Miscellaneous:**

&nbsp;

## Heuristics covered by Manual Review

**User Input and External Interaction:**

- Can I call functions with a non-existent identifier, 0 or a custom address that I control, and it doesn't revert but executes and returns > 0 values?
- External calls checklist
    - Reentrancy
    - DoS potential
    - Return values check
    - Gas forwarding
- Every Ether transfer implies potential code execution.
    - The receiving address can implement a fallback function that can throw an error. Thus, contracts should favor pull over push for payments.
- External calls can fail accidentally or deliberately.
    - To minimize external call failures, isolate each call in its transaction, initiated by the recipient, especially for payments. Avoid combining multiple send() calls in one transaction.
- When you are reviewing a multi-step process in a smart contract, you should ask yourself the following: Does the code verify at each step that the previous step has actually taken place, or has the developer assumed that the prior step has occurred without proper verification?
- Can user set their preferred slippage tolerance in a swap? If Yes, [user can set a zero slippage tolerance, and sandwich/frontrun themselves](https://youtu.be/zLnxRvf6IMA?t=4309) attack to extract the value of a collateral stuck in an unfavourable condition.
- When a function interacts with an AMM to swap user tokens. [Does the function check if the returned value can cover the users position](https://youtu.be/zLnxRvf6IMA?t=4309)?
- [Can the user force their position into a liquidatable state](https://youtu.be/zLnxRvf6IMA?t=4309)?
- After every external call does the function check if the state of the contract is impacted negatively?
- Is the price or [market condition the transaction was created, thesame with the price/market condition it is executed](https://youtu.be/zLnxRvf6IMA?t=5304)?
    - Is the transaction a two step transaction i.e. Is there seperate createOrder and executeOrder transaction?
    - Is the transaction executed in thesame block it was created?
    - Are transactions executed by keepers
    - Can I cause the latency of the oracle to increase?
- For protocol that [needs to liquidate a position or carry out similar action in a timely manner](https://youtu.be/zLnxRvf6IMA?t=6411)r
    - Can I prevent the liquidation from occurring by making the contract attempt to transfer a token to a blacklisted address.
    - Addresses that are blacklisted for popular ERC20 tokens such as USDC can be leveraged to exploit the exchange in a number of ways.
    - This could be an easy high find in most DeFi protocols, as ~70% of DeFi protocols have a liquidation functionality
- The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.
- Add buffer to BaseContract inherited by Proxies
- External call in assembly block([Solidity compiler STATICCALL](:/25143335f23248bcaf34e04ca8b9d0d5))
- [Assumption that the contract balance is equal to the deposits](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#assumption-that-the-contract-balance-is-equal-to-the-deposits)
- [Calling function multiple times with the same parameters](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#calling-function-multiple-times-with-the-same-parameters)
- [Arbitrarily long loops](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#arbitrarily-long-loops)
- [Improper state update when deleting items from a list](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#improper-state-update-when-deleting-items-from-a-list)
- [ETH / WETH handling](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#eth--weth-handling)
- Use of `from`, `to`, and `msg.sender` in one function or function chain
- Using raw contract balance (ETH or ERC20) for logic in action execution rather than accounting legimate business transactions and using that record.
- can I call functions with a non-existent identifier, 0 or a custom address that I control and it doesn't revert but executes and returns > 0 values?
- If [\_asset is ERC-777](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/abcf9dd8b78ca81ac0c3571a6ce9831235ff1b4c/contracts/token/ERC20/extensions/ERC4626.sol#L251), `transferFrom` can trigger a reentrancy [BEFORE the transfer happens](https://x.com/0xOwenThurm/status/1733179739542536529?s=20) through the `tokensToSend` hook. On the other hand, the `tokenReceived` hook, that is triggered after the transfer, calls the vault, which is assumed not malicious. Conclusion: we need to do the transfer before we mint so that any reentrancy would happen before the assets are transferred and before the shares are minted, which is a valid state.
- [DoS framework](https://youtube.com/shorts/nFxCSWM4AL8?si=I62N7pRmEfHabDM3)s:
    - Everytime you see a for-loop, ask if the iterable variable is bounded to a certain size.
        - If not, can a user add an arbitrary amount of items to that list? how much does it cost for the user to do that?
    - Look out for any external calls
        - Ask if there's a way for this calls to fail?
        - If they do fail, will that cause the top level external transaction to revert entirely?
        - If Yes, how can that actually affect the system? Significant negative impact or a negligible impact?
- Ways to make an external call fail
    - Sending ether to a contract that does not accept it
    - calling a function that does not exist
    - the external call execution runs out of gas
    - Third-party contract is simply malicious
- In ERC721, the \_safemint() function makes a callback to the to address parametermaking it vulnerable to reentrancy attacks, allowing the address to call back into thecode potentially causing unexpected behaviour.
- Take note if a ETH transfer function specifies a gas stipend like this: `payable(receiver).call{gas: 3000, value: amount}`. This 3000 gas might fall short for certain smart contract recipients needing more gas to receive ETH.
- don't forget an account/address without code in it (for example EOAs, wallets) will always return success on calling it with `call`, `staticcall` and `delegatecall`. Validating account/contract existence can sometimes save you from a catastrophic vulnerability
- Is there an external call in the modifier? [Reentrancy via modifier](https://medium.com/valixconsulting/solidity-smart-contract-security-by-example-03-reentrancy-via-modifier-fba6b1d8ff81)

&nbsp;

**Contract Design, Inheritance, and Execution:**

- Where are funds stored and where are funds retrieved from?
- Is this project using Chainlink on Arbitrum? If so, is there a sequencer uptime check?
- Can I change the "total" storage location without changing the individual storage locations, perhaps through a creative attack vector?
- Check for correct inheritance structure. (SWC-125)
- Emit an appropriate event for any non-immutable variable set in the constructor that emits an event when mutated elsewhere.
- Incorrect usage of using-for statement
- Check for gaps in the testing suite, especially lack of integration tests between two important contracts or very simplistic test cases of complicated processes. Focus on those areas and create more complicated test scenarios to probe the system into misbehaving.
- Does the test suite do anything different from the production code? Is the test suite calling functions that never get called in the protocol code?
- Once I have created a scenario where the system misbehaves, how can I leverage this to cause maximum damage?
- Is the protocol using the latest Solidity version or an older one? What common bugs are known for that specific version? Does this code utilize any similar patterns?
- What bugs were found in other contracts doing similar things or having similar dependencies -- use Solodit.
- State-changing calls that aren't verified
- System specification
    - Comprehensive system specification is crucial as it details how system components behave to meet design requirements.
    - It enables the evaluation of correctness during system implementation, ensuring alignment with design goals.
- Implicit requirements and assumptions should be flagged as dangerous.
- Some tokens such as USDT will revert if calling 'approve' when there is already a non-zero allowance
- 3 possible vulnerabilities when using signatures incorrectly:
    - No chain ID - cross-chain replay attack on an instance of your protocol on another chain
    - No nonce - signature replay attack
    - No domain (address of contract) - signature replay in another similar project
    - [Check that there's a frontrunning protection](https://youtu.be/pUWmJ86X_do?t=71767). Even with a nonce, ensure that the first sender of the transaction is the expected sender.
- If the system is setup in a way where there's no incentive for liquidating a position or the liquidation is not correctly enforced, the system can accumulate a lot of dust/insolvent positions which is a loss for the LP over time
- [Not following EIPs / Standards](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#not-following-eips--standards): there are often slight details (e.g., not reverting when one should revert or not following the specified rounding behavior) that are ignored by implementers.
- [Behavior when src == dst](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#behavior-when-src--dst)
- Calling a public/external function from within another function
- Contract business flows with a [sequence of multiple transactions](:/092db3a9e42c40fe90dba73529b6fb75)
    - And lacking atomicity
    - This exploit method (pre-empting a pending transaction belonging to an atomic business flow by paying a higher gas fee) is also called front running \[98\], whose root cause is usually atomicity violation.
- [Memory Safety Checklist](https://youtu.be/DRZogmD647U?t=36497)
    - Is the free memory pointer still accurate?
    - Did you overwrite that was previously allocated? (only change stuff after the FMP)
    - Is the zero slot, still a zero value
- Try-catch cases:
    - If the return data decoding throws an error it is not caught by the `catch` block. I[f an error happens during the decoding of the return data inside a try/catch-statement, this causes an exception in the currently executing contract and because of that, it i](https://docs.soliditylang.org/en/v0.8.23/control-structures.html#try-catch)s not caught in the catch clause. If there is an error during decoding of catch Error(string memory reason) and there is a low-level catch clause, this error is caught there.
- Many web3 devs are still not aware that the `PUSH0` opcode is still not supported on all of the L2s (e.g. Optimism and Arbitrum) and using pragma version 0.8.20 will be problematic.
- First thing you MUST check when you see a smart contract that has a `payable` function is if the native asset can be withdrawn from it. [Slither has a detector for this. Solidity developers should add it to their CI](https://github.com/marketplace/actions/slither-action)
- Good place to look for smart contract bugs - duplication of code and reimplementation of the same business logic. Often in such code there can be discrepancies/differences, where one part of the code has an important check which is missing in the other. [Possibly a Critical bug](https://solodit.xyz/issues/attacker-can-destroy-user-voting-power-by-setting-erc721powertotalpower-and-all-existing-nfts-currentpower-to-0-cyfrin-none-cyfrin-dexe-markdown)
- If `withdraw` has `whenNotPaused` modifier but the `deposit` function does not, that's a massive red flag.

&nbsp;

**State Change Security Considerations:**

- Stepwise jumps: [Frontrunning](https://youtu.be/DRZogmD647U?t=15261), [Link2](https://youtu.be/DRZogmD647U?t=18346)
    - Pay attention whenever there is a single transaction that adds value to something that is shared, in a single instance
- [Front Running Attack](https://youtu.be/DRZogmD647U?t=18451)
    - Identify variables that could impact the execution of a transaction.
    - Understand whether it involves a user's actions, such as creating an order, position, or swap, and what variables might influence these actions -- make a list.
    - Examine how someone could potentially engage in front-running to manipulate these variables, negatively affecting users.
    - MEV: for every function, ask: If someone sees this Tx in the mempol, how can they abuse that knowledge
- Always fuzz maths functions (including libraries)
- Malicious or compromised owners can trap contracts relying on pausable tokens
- Token should not allow flash minting
- Unused constructs
    - Any unused imports, inherited contracts, functions, parameters, variables, modifiers, events, or return values should be removed (or used appropriately) after careful evaluation.
- Ensure time-delayed change of critical parameters
- Not using (inheriting) an upgradable variant of an OpenZeppelin contract in an upgradable smart contract
- Exposing initialization functions: wrongly naming a function intended to be a constructor, the constructor code ends up in the runtime bytecode and can be called by anyone to re-initialize the contract.
- Check if contracts receiving ether have a withdraw or fallback function
- Check if parallel data structures are always in sync
- Check for Arrays that are too long to delete
- Check for jagged arrays - two arrays passed in that are not the same length
- 63 out of 64 rules
- Flash-Loanable ERC-721 tokens
- Gas Tokens to extract value from Gas Griefing
- Multicall functions with a payable modifier
- Using msg.value in a loop
- [Unbounded for loop](https://youtu.be/zLnxRvf6IMA?t=3724)
- Deleting a structs doesn't delete contained mapping or dynamic lists
- Subtractions that underflow and revert
- Down-casting can still overflow
- Parallel data structures
- Rounding errors
- A 'Multicall' function that works with 'address(this).delegatecall()' and using 'msg.value'. In a "delegatecall," the 'msg.value' is reused, hence it can be used for a double-spend exploit
- Storage updates in modifiers (except in a reentrancy lock)
- Updating the length of an array while iterating over it.
- Signatures not using EIP-712. (SWC-117 SWC-122)
- Updating a struct/array in memory won't modify it in storage.
- Magic numbers that are not constant or immutable
- Don't use `msg.value` if recursive delegate calls are possible (like if the contract inherits `Multicall`/`Batchable`).
- Not ensuring that a contract exists before a low-level call
- Using assembly for create2 -- (Prefer the modern salted contract creation syntax).
- Using assembly to access chainid or contract code/size/hash -- (Prefer the modern Solidity syntax).
- Upgradable contracts
- Function making a staticcall not marked as view in the interface
- Mock contracts
- No receive() function
- No fallback() function
- Write after write
- During an audit of a function that employs ecrecover, [it is ESSENTIAL to verify that the recovered address is not 0](https://x.com/bytes032/status/1729892994273219062?s=20).
    - This is crucial because ecrecover returns 0 for invalid signatures as well (such as when the value of v is neither 27 nor 28), enabling a malicious user to execute numerous undesirable actions. actions
- When you see ecrecover check for possible [Signature malleability](https://youtu.be/zLnxRvf6IMA?t=10421)
- Check for [Signature replay attack](https://youtu.be/jq1b-ZDRVDc?si=tvOERDDs9WQYjvsw) in Multisig Wallet; Same code different address; created by create2 has selfdestruct()
- [Code asymmetries](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#code-asymmetries): Symmetries between functions in projects, such as withdrawal undoing deposit state changes or deletion undoing additions, are crucial to avoid undesired behavior caused by asymmetries like forgetting to unset a field or subtract from a value.
- [Detection of uninitialized state](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#detection-of-uninitialized-state)
- [Forgetting to update the global state](https://github.com/OpenCoreCH/smart-contract-auditing-heuristics#forgetting-to-update-the-global-state)

&nbsp;

**Token-related Issues:**

- Vault not implementing EIP-4626: [Inflation attack](https://docs.openzeppelin.com/contracts/4.x/erc4626)
- 1: <=1 asset to shares (initial) exchange rate: [Inflation attack](https://docs.openzeppelin.com/contracts/4.x/erc4626)
- Watch out for rebasing tokens. If they are unsupported, ensure that property is documented.
- Watch out for fee-on-transfer tokens. If they are unsupported, ensure that property is documented.
- Oracle not checked for freshness
- Chain Incompatibility
- Hardcoded decimals
- Does it interact with an AMM (and does a swap)?
- No slippage control or high slippage tolerance
- A target contract for token approvals should not make arbitrary calls from user input.
- No staking fee
- No [Lock-up period](https://youtu.be/zLnxRvf6IMA?t=2960) associated with staking to receive a portion of the reward pool
- Unchecked return value of Send, transfer, transferFrom, low level call
- If there is \`token.transferFrom\` in the contract, users must not manage the \`from\` parameter. Otherwise hacker can take advantage of other user’s appovals and rob them! In 99% of cases \`from\` should be just \`msg.sender\`.
- What are the different assets/tokens in the contract and how are the accounting for each type handled? Money and asset flow for each asset type.
    - Ensure ERC tokens and Native network tokens are handle with the appropriate balance, transfer, send functions.
- USDT does return a boolean on its transfer functions


&nbsp;

**Miscellaneous:**

- Solidity version less than 0.8.0
- Catch block actually catches return value -- catch (bytes memory returnData) {} (ref: https://youtu.be/DRZogmD647U?t=5380)
- Modifying an array of unknown size that increases in size over time can lead to such a Denial of Service condition.
- Immutable values are not maintained on upgrade since these values are a part of the contract code
- Watch out for tokens that use too many or too few decimals. Ensure the max and min supported values are documented.
- A target contract for token approvals should not make arbitrary calls from user input.
- No sanity checks to prevent oracle/price manipulation.
- Cyclomatic complexity > 11
- [<ins>Missing Protection against Signature Replay Attacks</ins>](https://kadenzipfel.github.io/smart-contract-vulnerabilities/vulnerabilities/missing-protection-signature-replay.html#missing-protection-against-signature-replay-attacks)
