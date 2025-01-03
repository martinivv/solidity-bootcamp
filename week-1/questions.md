# Why was ERC1363 introduced, and what issues are there with ERC777?

**1. ERC-1363:**

- An extension interface for ERC20 tokens;
- Backwards Compatible;
- Enables executing code on a recipient contract after `transfer` or `transferFrom`;
- OR code on a spender contract after `approve`;
- Single transaction;
- Eliminates the need for pre-approving allowances and **reduces** the risk of granting excessive permissions;
- `approveAndCall` and `transferAndCall` have the same transaction racing risk as `approve` and `transferFrom`, allowing misuse of both old and new allowances due to ordering issues.

**2. ERC-777:**

- Introduced in November, 2017 as an extension over ERC20;
- Backwards Compatible with 20 tokens;
- Getting a transfer hook called, requires registering in ERC-1820 registry;
- Quite gas consuming;
- Attack vectors for reentrancy [attacks](https://blog.openzeppelin.com/exploiting-uniswap-from-reentrancy-to-actual-profit).

<br>

# Why does the SafeERC20 exist and when should it be used?

The **ERC-20 standard** only states that the token should throw an error if the user tries to transfer more than their balance. However, if the transfer fails for some other reason, then the standard does not explicitly state what should happen.

In practice, ERC-20 tokens have been implemented in inconsistent ways: some reverting on failure, and others not returning any Boolean at all (i.e. not respecting the function signature).

The library **SafeERC20** handles both kinds of ERC-20 tokens. Specifically, it makes a transfer call to the address, and:

- If a revert happens, SafeERC20 bubbles up the revert. This handles tokens that revert on failure, but don’t necessarily return a Boolean.
- If there is no revert, it checks whether data was returned at all
  - if no data was returned and the token address turns out to be an empty address rather than a smart contract, the library reverts.
  - if data was returned, and the return is a false value, then SafeERC20 reverts.
- Otherwise, the library does not revert, signaling a successful transfer.
