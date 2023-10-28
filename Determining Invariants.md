# Determining Invariants

## Contract
- Write down and test invariants about relationships between stored state.

## Functions
- Write down and test invariants about state before a function can run correctly.
- Write down and test invariants about the return or any changes to state after a function has run.

&nbsp;
## Handy Guide

https://x.com/bytes032/status/1706665676839272489?s=20

How to determine all the invariants of the function you're examining:

- Take into account all possible execution paths.
- Consider which state leads to specific paths.
- Define nodes for "when state is x".

All the statements starting with "it should" are your invariants.
