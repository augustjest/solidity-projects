https://github.com/0xMacro/student.augustjest/tree/4f0492163d5c55d8aae14067a12956439bf65674/ico

Audited By: Alex.S

# General Comments

This is a very good solution to the exercise. There are a few points below, but most of them are not big enough to score points. Your readme is great, clearly explaining your thoughts, and your test coverage is good too. 

# Design Exercise

Some interesting thoughts here. I think a solution could simply be to record the total contribution of each address and have redemption initiated by the contributor (pull rather than push model) over as many calls as they wish, but limited to to a linearly increasing fraction of the total value. With this fraction increasing to 1 over some fixed period after the start of the open phase.

# Issues

**[M-1]** Anyone can turn tax on or off

The `setTax` function in `SpaceCoinICO.sol` is only callable by the owner, but the `setTax` function in `SpaceCoin.sol` can be called by anyone. 

**[L-1]** Dangerous Phase Transitions

If the 'advance' function is called twice, a phase can accidentally 
be skipped. There are a few situations where this might occur:

1. Front-end client code malfunction calling the function twice.
2. Human error double-clicking a button on the interface on accident.
3. Repeat invocations with intent - Uncertainty around whether or not a 
transaction went through, or having a delay before a transaction processes, 
are common occurrences on Ethereum today.

Consider refactoring this function by adding an input parameter that 
specifies either the expected current phase, or the expected phase to 
transition to.

**[Extra Feature]** Removing seed investors

As you said in your readme, this is extra functionality. It also raises the possibility of someone being a seed investor, making a seed investment, and then being removed from the seed investor list. They would then not be able to make further seed investments, but they would still be a seed investor, whereas anyone looking at the `isSeedInvestor` mapping would be told that they were not. In smart contracts every extra feature, and every extra line of code, increases your exposure to risk. Always keep things as simple as possible. 

**[Q-1]** Immutable values are using contract storage

If you have values which are set in your contract constructor and then never changed, as `owner` and `treasury` are in both `SpaceCoinICO` and `SpaceCoin`, then you can declare them with the `immutable` keyword. This will save gas as the compiler will not reserve storage for them and instead inline the values where they are used.

**[Q-2]** Error messages are quite long

The error messages in your `require` statements are quite long. Whilst this makes the nature of the error very clear to both users and readers of the contract, it does consume more gas. Consider either using shorter error messages or switching to custom errors with `revert` (see https://blog.soliditylang.org/2021/04/21/custom-errors/)

**[Q-3]** Public functions only called externally

The functions `advance`, `setPausedState`, `addSeedInvestors`, `removeSeedInvestors` and `redeem` are declared public when they could be `external` as they are never called from within the contract. Consider using `external` instead to reduce contract size a little and make the intended usage clearer.

**[Q-4]** Unnecessary copying of calldata values to memory

The functions `addSeedInvestors` and `removeSeedInvestors` each take an array of addresses declared as `memory`. As a result these arrays will be copied into memory from calldata. If they were declared as `calldata`, and the functions themselves declared `external` rather than `public`, then this copying could be avoided.

**[Q-5]** `assert` mixed in with `revert`

In `_checkValidityOfContribution` the current contributions of the message sender are compared with the limits for the current phase. In the case that we are in the seed phase and the contributions exceed the seed phase limit an assertion fails. Similar cases in the general and open phases are handled by a revert. It would be better to be consistent in the level of error generated by using `revert` in every case.

**[Q-6]** Unnecessarily complicated code for contribution handling

The code to check that the amount of a contribution does not exceed the currently applicable limits is split across the `contribute` function and its `_checkValidityOfContribution` modifier, which makes the logic harder to follow. This code is complicated by the fact that it handles refunds to contributors who are permitted to make some contribution but send more than is allowed. This is not necessary, simply rejecting a contribution that would exceed the limits would enable the code to be simplified and would still meet the requirements of the specification. Always favour simple and more obviously correct code over support for nice to have, but not strictly necessary, features.

# Score

| Reason                     | Score |
| -------------------------- | ----- |
| Late                       | -     |
| Unfinished features        | -     |
| Extra features             | 1     |
| Vulnerability              | 3     |
| Unanswered design exercise | -     |
| Insufficient tests         | -     |
| Technical mistake          | -     |

Total: 4

Good job!
