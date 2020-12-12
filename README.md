# Indexed Finance smart contracts security review by Mudit Gupta

## Objective of the review

The review's focus was to verify that the smart contract system is secure, resilient, and working according to its specifications. The review activities can be grouped in the following three categories:

**Security**: Identifying security related issues within each contract and the system of contracts.

**Sound Architecture**: Evaluation of this system's architecture through the lens of established smart contract best practices and general software best practices.

**Code Correctness and Quality**: A full review of the contract source code.

This review was initially based on commit hash `84aed90c86481f75c50911a139e5ef38fd19d4fb` and then updated for commit hash `822e067a96f599103753376cd03dde856849f703` of <https://github.com/indexed-finance/indexed-core/>

## Findings

Overall, the code follows high-quality software development standards and best practices.

During the review, 0 Critical, 1 Major, 3 Minor, and 6 Informational issues were found.

- A critical issue represents something that can be relatively easily exploited and will likely lead to loss of funds.
- A major issue represents something that can result in an unintended behavior of the smart contracts. These issues can also lead to loss of funds but are typically harder to execute than critical issues.
- A minor issue represents an oddity discovered during the review. These issues are typically situational or hard to exploit.
- An informational issue represents a potential improvement. These issues do not pose any practical security risks.

### Major

##### 1.1 `denorm` weight is incorrectly assumed to be equal to minimum weight at certain places

##### Description

In the `gulp` function and the `_updateBalanceIn` function of BPool, `denorm` weight of uninitialized tokens is assumed to be equal to the minimum weight. However, it can be 1.33 times of the the minimum weight in reality. This discrepancy will cause the system to undervalue that token and the users will be able to get favourable swaps.

##### Status update

This issue has been fixed. The `denorm` weight is now set based on the balance of the contract.

### Minor

#### 2.1 Hardcoded function selectors are used instead of interfaces and objects

##### Description

The codebase uses hardcoded function selectors like `TRANSFER_FROM_SELECTOR` to do low level calls instead of creating an IERC20 object and calling the function via the high level language. This prevents the compiler from doing static checks on the function selector and can result in typos. It's recommended to use the high level language to do such calls.

##### Status update

This issue has been fixed.

#### 2.2 Inefficient sorting

##### Description

In `orderCategoryTokensByMarketCap`, reading all values directly from `_categoryTokens` and then sorting them will be cheaper than taking `orderedTokens` as input and verifying the order since the number of tokens is low. I'd recommend using insertion sort for this. This change will also make the code more legible.

##### Status update

This issue has been fixed.

#### 2.3 Missing address validation checks

##### Description

The constructor of `Owned.sol` is missing the zero address check. The check should be added to prevent unintentional human errors.

##### Status update

This issue has been fixed.

### Informational

#### 3.1 Division before multiplication

##### Description

Doing division before multiplication can cause loss of some precision since Solidity truncates intermediate results and does not handle floating points.

L138-140 of `PoolController`, should be modified to do the multiplication of `totalValue` before the division with `25`.

```
minimumBalances[i] = prices[i].reciprocal().mul(
    totalValue / 25
).decode144();
```

#### 3.2 Assembly code is used excessively.

##### Description

There are some places where assembly code is required because it's simply not possible to do the required thing in Solidity. However, at many places, assembly code is used for minor gas savings. Assembly code is harder to review and the cost vs benefit should be analyzed before using it.

#### 3.3 `uint256` being used to represent a `boolean`

##### Description

In L250 of `BPool`, `uint256[] memory receivedIndices = new uint256[](tLen);` is defined but `receivedIndices` is used as a boolean with only 0 and 1 as the possible values. It's recommended to use a boolean to make the intentions clearer.

#### 3.4 Unnecessary copying of data

##### Description

In L272 of `Bpool`, `Record memory record = records[i];` is defined but it's not needed as `records[i]` can be directly used where `record` is being used.

#### 3.5 Code duplication can be reduced

##### Description

There are a few single and batch functions like `addToken` / `addTokens` that have plenty of duplicated code among them. It's recommended to factor out the common code in an internal/base function.

#### 3.6 IPool should be renamed to something else

##### Description

It's common practice to name interfaces as I**** in Solidity. Naming logic contracts like this adds to confusion. It's recommended to rename `IPool` to something else.

**NOTE**: Most of the suggestions from the informational issues have been implemented already.

## Static analysis

Slither was used to do static analysis of the contracts and the output of slither can be found at <https://gist.github.com/maxsam4/f6d0a55e41721512bafb917040201f7e>

**NOTE**: Please keep in mind that most of the issues flagged by Slither are false positives.

## Disclaimer

This report is not an endorsement or indictment of any particular project or team, and the report do not guarantee the security of any particular project. This report does not consider, and should not be interpreted as considering or having any bearing on, the potential economics of a token, token sale or any other product, service or other asset. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty. This report does not provide any warranty or representation to any Third-Party in any respect, including regarding the bugfree nature of code, the business model or proprietors of any such business model, and the legal compliance of any such business. No third party should rely on this report in any way, including for the purpose of making any decisions to buy or sell any token, product, service or other asset. Specifically, for the avoidance of doubt, this report does not constitute investment advice, is not intended to be relied upon as investment advice, is not an endorsement of this project or team, and it is not a guarantee as to the absolute security of the project. I owe no duty to any Third-Party by virtue of publishing these Reports.

The scope of my review is limited to a review of Solidity code and only the Solidity code noted as being within the scope of the review within this report. The Solidity language itself remains under development and is subject to unknown risks and flaws. The review does not extend to the compiler layer, or any other areas beyond Solidity that could present security risks. Cryptographic tokens are emergent technologies and carry with them high levels of technical risk and uncertainty.

This review does not give any warranties on finding all possible security issues of the given smart contracts, i.e., the evaluation result does not guarantee the nonexistence of any further findings of security issues. As one review cannot be considered comprehensive, I always recommend proceeding with several independent reviews and a public bug bounty program to ensure the security of smart contracts.
