# Submission Template

**Title:** *&lt; ROOT CAUSE&gt; + &lt; IMPACT &gt;*  
e.g. &lt; s\_owner not being set to immutable&gt; + &lt; wastes gas&gt;

&nbsp;  
**Severity:**  
[*reference*](https://docs.codehawks.com/hawks-auditors/how-to-evaluate-a-finding-severity)

&nbsp;
**Relevant GitHub Links:**
- code line link

&nbsp;  
**Finding:**

## Summary

*What's wrong with the issue raised explain as succinctly as possibly.*  
e.g.:  
The `PasswordStore::s_owner` variable is unable to be changed, and therefore should be immutable to save gas.

&nbsp;

## Vulnerability Details
If a variable never changes, then reading from storage is less gas efficient, and instead we should read from the bytecode. Using an immutable variable.

```diff
- address private s_owner;
+ address private immutable s_owner;
```

&nbsp;

## Proof of Concept

The scripts and test suite below demonstrate the validity and severity of the vulnerability.

***
---

>*This section is for situations where a script is required to complete the PoC, as the test alone won't suffice.*

### How to Run the Scripts

**Requirements**
- Install [Foundry](link)
- Clone the project codebase to your local workspace.
- Copy the scripts below into the script folder. Note the file names provided in the code.
- Create a .env file.
- Copy the following variables and replace "InsertYourData" with your corresponding details:

```env
ALCHEMY_PRIVATE_KEY=InsertYourData
ALCHEMY_GOERLI_URL=https://InsertYourData
ALCHEMY_API_KEY=Insert-Your-Data
ETHERSCAN_API_KEY=InsertYourData
```

**Step-by-step Guide to Run the PoC**
1. Ensure the above requirements are installed.
2. Enter `anvil` in a new terminal to set up the Anvil network locally (copy one of the private keys for your environment variable).
3. Run `source .env` to load .env variables into the terminal.
4. Run the necessary command to deploy the contracts.
5. Run the command to check the network state before the exploit, and take note of the current state.
6. Run the command to run the exploit script.
7. Run the command again to see the state difference and compare it with the result of step 5.

### Codebase

The code below consists of Foundry scripts that deploy the contract to the local Anvil (Sepolia) network and interact with it through our exploit script.

**Deployment Script**
```solidity
// File name:
// Code goes here
```

**Exploit Script(s)**
An overview of what the code does. Indicate what the judge should focus on.

```solidity
// File name:
// Code goes here
// Remember to add comments explaining what the code does
```

**Proof of Exploit**

What changed after running the script:

Protocol State before exploit:
Protocol State after exploit:
```solidity
// File name:
// Code goes here
```

---
***

### How to Run the Test

**Requirements**
- Install [Foundry](link)
- Clone the project codebase into your local workspace.
- Copy the tests from the Codebase section below into the test folder. Note the file names provided in the code.

**Step-by-step Guide to Run the Test**
1. Ensure the above requirements are met.
2. Run the necessary command to test `testFunctionExample()`.
> *Note: Read the test function comments to understand all the cases being tested.*

### Codebase

The codebase below utilizes Foundry it tests.

**Test Cases**
```solidity
// Code goes here
// Remember to add comments explaining what the code does
```

## Impact

### Implications
Passing the above tests implies that the vulnerability:
- Allows access to ____.
- Changes ____.
- Corrupts ____.
- Withdraws ____.
- ...

###Exploit Scenario
John deposits $1000, and then blah, blah, blah...

&nbsp;

## Tools Used
- Foundry
- Slither
- etc.

&nbsp;

## Recommendations

To fix this and that, follow these steps:

```diff
- Old code // Instead of this
+ New code // Do this!
```

&nbsp;
&nbsp;
* * *

## Real Example

High Risk Findings on Canto (Code4rena)
***

##  \[Title\] User doesn’t have to deposit for a week into the market to get their weekly reward from the `LendingLedger`</ins>

In the `LendingLedger` contract, a user is rewarded with CANTO tokens depending on how long he has his deposit in the market. Rewards are distributed for each week during which the deposit was inside the market. However, the user can cheat this condition because we are rounding down to the start of the week, so the user can deposit at 23:59 at the end of the week and withdraw at 00:00 and still get rewarded as if he had his deposit for the whole week.

&nbsp;
### Proof of Concept
Test case for the `LendingLedger.t.sol`
```solidity
      function setupStateBeforeClaim() internal {
        whiteListMarket();
        vm.prank(goverance);
        ledger.setRewards(0, WEEK*10, amountPerEpoch);
        // deposit into market at 23:59 (week 4)
        vm.warp((WEEK * 5) - 1);
        int256 delta = 1.1 ether;
        vm.prank(lendingMarket);
        ledger.sync_ledger(lender, delta);
        // airdrop ledger enough token balance for user to claim
        payable(ledger).transfer(1000 ether);
        // withdraw at 00:00 (week 5)
        vm.warp(block.timestamp + 1);
        vm.prank(lendingMarket);
        ledger.sync_ledger(lender, delta * (-1));
    }
    function testClaimValidLenderOneEpoch() public {
        setupStateBeforeClaim();
        uint256 balanceBefore = address(lender).balance;
        vm.prank(lender);
        ledger.claim(lendingMarket, 0, type(uint256).max);
        uint256 balanceAfter = address(lender).balance;
        assertTrue(balanceAfter - balanceBefore == 1 ether);
        uint256 claimedEpoch = ledger.userClaimedEpoch(lendingMarket, lender);
        assertTrue(claimedEpoch - WEEK*4 == WEEK);
    }
```

&nbsp;
### Tools Used
Foundry

&nbsp;
### Recommended Mitigation Steps

It’s difficult to propose a solution for this exploit without major changes in the contract’s architecture. Perhaps we can somehow split the amount based on the time the sync was made inside the week, let’s say Alice’s `last_sync` was in the middle of week0, she deposited 1 ether, thus her amount for the current epoch will be 1/2 ether. However there is a caveat, how do we fill the gaps? We can’t fill them with 1/2 ether. We can use this struct though,

```
Amount {
    uint256 actualAmount,
    uint256 fraction
}
```

so we can use `fraction` for the current epoch and `actualAmount = 1 ether` to fill the gaps.

&nbsp;
*<ins>alcueca (Judge) increased severity to High and commented</ins>:
> Chosen as best due to clarity, conciseness, and presence of executable PoC*
