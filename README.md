[Arcade.xyz](https://docs.arcade.xyz/docs/faq) facilitates trustless borrowing, lending, and escrow of NFT assets on EVM blockchains. This repository contains a community-focused extension to the core lending protocol, which allows for on-chain registration and enumerability of vault contents.

# Relevant Links

- 🌐 [Website](https://www.arcade.xyz) - Our app website, with a high-level overview of the project.
- 📝 [Usage Documentation](https://docs.arcade.xyz) - Our user-facing documentation for Arcade and the Pawn Protocol.
- 💬 [Discord](https://discord.gg/arcadexyz) - Join the Arcade community! Great for further technical discussion and real-time support.
- 🔔 [Twitter](https://twitter.com/arcade_xyz) - Follow us on Twitter for alerts and announcements.

# Disclaimer

__*Arcade.xyz/Non-Fungible Technologies, Inc. does not actively maintain these contracts*__. These contracts are strictly external to the core lending protocol and only for use by integrators. Non-Fungible Technologies makes no claims as to the correctness of these contracts, or what security considerations must be made for integrators who wish to use these contracts. Integrators are welcome to fork, modify, and deploy their own versions of these contracts.

# Overview of Contracts

See natspec for technical detail.

### VaultInventoryReporter

The `VaultInventoryReporter` contract contains a registration system for on-chain assets stored in an [Arcade.xyz Asset Vault](https://github.com/arcadexyz/v2-contracts/blob/main/contracts/vault/AssetVault.sol). Traditionally, vaults are generalized wrappers that can secure any asset sent to the contract: this means that deposits can be permissionless, and registration of vault contracts must take place separately.

The reporter contract, via the `add` function, allows new items to be registered, as long as they are owned by the vault. Registrations are unique to a given combination of token address and token ID (ERC20s are stored such that `tokenId == 0`).

Once items have been registered, the registered items can be enumerate via the `enumerate` or `enumerateOrFail` functions. The former will return all previous registrations on a vault, whereas the latter will do the same, in addition to verifying in real time that the reported items are still owned by the vault. For most use cases which rely on enumeration, `enumerateOrFail` works better, since `enumerate` __may return stale items__.

Registrations can also be removed or cleared via the eponymous functions.

Only vault owners or approved contracts are allowed to perform modifications on a vault's registration. Individual vault owners can approve third party addresses either via on-chain approval or permit functionality (see `addWithPermit` and others). In addition, the contract owner can globally approve addresses to register items to any vault - this is useful for more complex integrations (see `VaultDepositRouter`).


### VaultDepositRouter

The `VaultDepositRouter` contract combines registration with deposits to the Arcade.xyz Asset Vault. This allows deposits and registration to happen in a single transaction, which reduces registration overhead and UX friction. Note that for the router to be used, it must be globally approved by the relevant `VaultInventoryReporter`. Each deposit router is linked to a specific Asset Vault implementation (the `factory`), and a specific reporter (the `reporter`).

The router only allows vault owners or approved adresses for a given vault ownership key to use the deposit router, ensuring that unauthorized registrations cannot take place (unauthorized deposits can always take place via direct on-chain transfers). The router has no permissions and holds no state.