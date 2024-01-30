# Security_Auditors_Starter_Pack
- [Miro board](https://miro.com/app/board/uXjVMgF6YT4=/?share_link_id=15940391732)
- [Whimsical board](https://whimsical.com/public-home-RCh9khB7SdtPeBczNF5DjS)
- [Google Sheet Model](https://docs.google.com/spreadsheets/d/1LoUfjXRslmXcaoZVNV3Ya1btsAshg5V3ZkymYcunZPs/edit?usp=sharing)
- [Twitter List](https://‪https://x.com/i/lists/1699739698728407461)

## General Overview



This repository serves as a guide to help you develop your own unique methodology. Those who have attempted to implement this system as it stands can attest to its time-consuming nature. The repository is intentionally kept broad to accommodate various possible approaches you can derive from it.

## Extracting the Most from the Repository

To make the best use of this repository, consider the following tips based on our experience:

- Utilize the "redflag" and "common pitfalls" files as paired mappings. Instead of manually checking a codebase for bugs using "common pitfalls" and vulnerabilities (which is impractical), run the "redflag" file through the codebase. Create a list of keywords or patterns found in the codebase and cross-check them with the "common pitfalls" table to identify potential vulnerabilities. This process generates a bug lead list for further assessment. To enhance efficiency, we aim to consolidate all files related to common vulnerabilities into one, combining "common pitfalls" and...

- Combine the codebase, Miro board, and Google Sheet model for a comprehensive understanding of the codebase, facilitating code walks, reviews, and test suite development. To minimize time spent switching between webpages, we have integrated most of the first step and key aspects of other steps into the Miro board and Google Sheet model. For example, "Perspective," which helps you gain new insights into the codebase, can be found in the latest Miro workspace.

## Repo Content

Audit Methodology.md (Partly integrated in the Miro Board)
Audit Question Formulation Strategies.md: Help you formulate audit questions. Asking good questions is a good way to better understand a project
Determining Invariants.md (Integrated into the Google sheet model)
Gas Optimization
Manual review approaches.md (Integrated into the Miro Board)
Redflags & Code Smells.md
Security Common Pitfalls.md
Stakeholder Analysis.md (Integrated into the Miro Board)
Submission Template.md


## Immediate To-Dos

We welcome your support to further develop this repository. Ways to contribute include:

- Adding more "red flags" and "code smells" (Note: A "red flag" should directly relate to a vulnerability, not speculations or assumptions, e.g., Accepted red flag: A function not following the check-effect-interaction pattern. Unaccepted red flag: Finding // TODO comments in "red flags").
- Recognizing that we haven't always strictly adhered to our rules, it's crucial to proofread the repository and rectify any oversights and entries that don't align with our guidelines.
- Incorporating new relevant files, list items, comments, and extensions.

## Future Targets

Our future goal is to transform the "Redflags & Code Smells.md" into a static analyzer. To achieve this, we are focused on collecting and aggregating as many "redflags" and "code smells" and "vulnerabilities" as possible. 
> [Version 0.1.0](https://github.com/Renzo1/RenZo_Scanner) is out now. Spoiler alert: It's pretty basic at the moment ;)
