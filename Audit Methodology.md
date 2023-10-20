# My Methodology

> *Sourcegraph: Search with the relevant keywords to find if the potentially to-be-audited repository is already public*

> *Terminal SetUp for Testing: Chisel terminal; Anvil terminal; Command terminal*

> *Time Planning  
> Step 1: 5% (Prepare before start date to make this faster)  
> Step 2 & 3: 10%  
> Step 4: 40%  
> Step 5 & 6: 40%  
> Step 7 & 8: 5%*

## Step 1: Deep Dive

### Understand Project Goals:

- Read the project's whitepaper and documentation to understand the smart contract's intended purpose and functionality.
- List out the high level goals of the system (Use this for the [Miro template](https://miro.com/app/board/uXjVMgF6YT4=/?share_link_id=281957613648))
- Engage with the development team or stakeholders to clarify objectives and functionalities.
- Diagrams the system - External API, State transition, Flow of funds
- Study the underlying network for the protocol
    - It's shortcomings, pitfalls, loopholes, known exploit, and how the audited projects handles all of that
    - Read the networks documentation and all GitHub (community) discussions

### Stakeholder Analysis:

- This has been incorporated into the  [Miro template](https://miro.com/app/board/uXjVMgF6YT4=/?share_link_id=281957613648). 
- However, you can check [Stakeholder Analysis Template](:/033bafdfec9c460ab7b2519787b5a564) for the detailed version

#### Build a List of Initial Questions:

- This has been incorporated into the  [Miro template](https://miro.com/app/board/uXjVMgF6YT4=/?share_link_id=281957613648). 
- Check out [Audit Question Formulation Strategies](:/ba40642685f5488bb9454487eb9034fa)

### Ecosystem System Analysis
Go into the Project Community and see what the trending topics are:
- You might find a unique issue from what the community is still discussing or divided on.

### High-level Business Logic Simulation (Optional)

- Validate the Business logic by simulating it with Machinations (or other virtual economy simulators).
- Recommended for complex systems and when they is ample time for the audit.

&nbsp;

## Step 2: Run Automated Tools 
> *Todo: add links to the tools folders, containing tutorials, in joplin*

The goal here is to find all the low hanging fruits, and get them out of the way early.

| Tool | Tool Type | Description | When to Use | How to Use |
| --- | --- | --- | --- | --- |
| Halmos | \-  | Halmos is a symbolic testing tool for EVM smart contracts. | \-  | \-  |
| Securify2.0 |     | is a security scanner for Ethereum smart contracts supported by the Ethereum Foundation and ChainSecurity. |     |     |
| **Mythx** (paid) |     | A security analysis platform for Solidity smart contracts, combining static and dynamic analysis to detect vulnerabilities and generate detailed reports. |     |     |
| Slither | [Static Analyzers](https://book.getfoundry.sh/config/static-analyzers#static-analyzers) | This static analysis tool examines Solidity source code for security vulnerabilities and checks compliance with best practices. Can be integrated with Foundry |     |     |
| Mythril | [Static Analyzers](https://book.getfoundry.sh/config/static-analyzers#static-analyzers) | A bug-hunting framework that helps to identify potential vulnerabilities in Solidity smart contracts. |     |     |
| 4naly3er | Static Analyzers | stands as a pivotal static analysis tool deployed in the auditing of smart contracts, particularly prevalent in the Code4rena platform for automated smart contract auditing. |     |     |
| Semgrep |     | is a fast, open source static analysis tool for finding bugs, detecting vulnerabilities in third-party dependencies, and enforcing code standards. |     |     |
| Codex |     | Use [codex](https://openai.com/blog/openai-codex/) to find vulnerabilities |     |     |
| \### Special Case: |     |     |     |     |
| Diffusc |     | uses differential fuzzing to simplify the process of reviewing smart contract upgrades. It’s hardly a tool for day-to-day operations but may prove useful when comparing two smart contract implementations. |     |     |

&nbsp;

## Step 3: Initial Code Review:
Spot any problem areas needing attention before the in-depth review begins. Check out [Miro template](https://miro.com/app/board/uXjVMgF6YT4=/?share_link_id=281957613648). 
 
### Source Code Analysis (Pt I)
- Construct a mental model of what you expect the contracts to look like before checking out the code (if possible).
- Glance over the contracts to get a sense of the project's architecture. Tools like Surya can come in handy.
- Compare the architecture to your mental model. Look into areas that are surprising.
- Create a [Threat Model](:/8aa5bfd775dd400faf44f7a76c71488c) and make a list of theoretical high level attack vectors.
- Look at areas that can do value exchange. Especially functions like `transfer`, `transferFrom`, `send`, `call`, `delegatecall`, and `selfdestruct`. Walk backward from them to ensure they are secured properly.
- Look at areas that interface with external contracts and ensure all assumptions about them are valid like share price only increases, address can receive ether etc.

### Source Code Analysis (Pt II)

- Walk through each of the high level goals in the code and walk the code path and try to understand from a high level exactly how this goal is achieved
- Map out the every code path for each functions -- [Each code path should have its own unit test.](https://book.getfoundry.sh/tutorials/best-practices#best-practices-1)
- Build a list of [invariants](:/f6bc6ae3247741d78105c4bebd925dc4) and other things that would be very harmful for the protocol if they broke
- Analyze the code's structure, modules, dependencies, and coding style.
- Check code comments and documentation for clarity, correctness, and completeness.
- Evaluate the development environment for any potential compilation problems, executes the provided tests, and verifies both the functional and non-functional requirements.
- Run the codebase against [Red-flag Alerts](:/82edb44b30014512a4480d2bed42e8b1), [✅Top Common Vulnerabilities](:/092db3a9e42c40fe90dba73529b6fb75), and [✅The Solcurity Standard](:/17db19daaf4a49d4b47c1f3effc16992)
- Leave @audit tags above and beside suspicious code sections (knobs)

#### Add to Question List

[Audit Question Formulation Strategies](:/ba40642685f5488bb9454487eb9034fa)

### Visual Tools

- **Solgraph**: A tool generating a DOT graph, visualizing the function control flow of a Solidity contract and highlighting potential security threats.
- **Surya**: offers detailed contract analysis and visualization, especially suitable for complex contract interactions.

> Note: Solidity Visual Developer (VSCode Plugin) uses Surya  
> Tip: Use visual tools to catch functions not in use

&nbsp;

## Step 4: Line-By-Line Review

Let the audit begin!

### Find & document issues

An exhaustive line-by-line review meticulously examining each segment of the code for all possible vulnerabilities, including:Issues outlined in the SWC registry, Data manipulations, Access violations, Flash loans, Complex vulnerabilities emerging from contracts interactions. See [Security Pitfalls Checklists](:/0655cc538dd54e44b6aa17bd207dbf31)

- Conduct reviews from various perspectives including:
    - Follow the money flow.
    - Identify potential system-halting DoS attacks.
    - Review from the perspective of every actor in the threat model.
    - Review the code with these \[approaches\]([Manual review approaches](:/517d5fa541bc4c76bd7fc8df2a7f4992))
- Glance over the project's tests + code coverage and look deeper at areas lacking coverage. (handy tools: [Coverage Gutters](:/7b96bf7c6d0c4cb59b3881e5916389d5), Foundry)
- Run Automated tools like Slither/Solhint and review their output.

### Comparative Analysis and Industry Standards:

- Look at related projects and their audits to check for any similar issues or oversights.
- Conduct a comparative analysis with similar projects to identify design differences and their reasoning (e.g. business logic differences).
- Investigate industry-specific design standards and assess the contract's alignment with them.

### Go back to @audit tags for deeper review

Resolve all the issues raised in the initial code review phase.

### Answer all initial questions raised

Answer all the question on the question list.

&nbsp;

## Step 5: Testing

If you have a complex utility function with low-level assembly, consider writing a foundry fuzz test (Differential Testing) to compare it with a widely-used utility function or use SMTChecker like Halmos for formal verification.  
[📙Audit Testing With Foundry](:/bb99225262074d0e9782d0f48eda7e15)

- [Unit Tests](https://book.getfoundry.sh/tutorials/best-practices#best-practices-1)
    
- [Integration Tests](https://book.getfoundry.sh/tutorials/best-practices#best-practices-1)
    
- [Forked Tests](https://book.getfoundry.sh/tutorials/best-practices#best-practices-1)
    
- Fuzz Testing
    
    - Stateless Fuzzing
    - Stateful Fuzzing
        - Open Testing
        - Continue On Revert Testing
        - Fail On revert Testing
        - Differential Testing
- What are some edge cases for the tested function
    
- Utilize threat model Include adversarial tests
    
- Take advantage of @audit notes (knobs) written in earlier steps
    
- Always reference [Foundry Best Practices](https://book.getfoundry.sh/tutorials/best-practices.html)
    

&nbsp;

## Step 6: Writing PoCs

- Write PoCs for every potential exploits found

&nbsp;

## Step 7: Peer Review

Compare and contrast by debating and discussing findings with co-auditors

&nbsp;

## Step 7: Report

After an intensive cross-review of the findings, formulates a detailed final report covering crucial issues, vulnerabilities, and executed tests.
