# Substrate Blockchain Treasury Setup

This document guides you through the process of integrating and configuring the Treasury pallet into your Substrate-based blockchain.

### Build

Use the following command to build the node without launching it:

```sh
cargo build --release
```

### Embedded Docs

After you build the project, you can use the following command to explore its
parameters and subcommands:

```sh
./target/release/node-template -h
```

You can generate and view the [Rust
Docs](https://doc.rust-lang.org/cargo/commands/cargo-doc.html) for this template
with this command:

```sh
cargo +nightly doc --open
```

### Single-Node Development Chain

The following command starts a single-node development chain that doesn't
persist state:

```sh
./target/release/node-template --dev
```

To purge the development chain's state, run the following command:

```sh
./target/release/node-template purge-chain --dev
```

To start the development chain with detailed logging, run the following command:

```sh
RUST_BACKTRACE=1 ./target/release/node-template -ldebug --dev
```

Development chains:

- Maintain state in a `tmp` folder while the node is running.
- Use the **Alice** and **Bob** accounts as default validator authorities.
- Use the **Alice** account as the default `sudo` account.
- Are preconfigured with a genesis state (`/node/src/chain_spec.rs`) that
  includes several prefunded development accounts.

To persist chain state between runs, specify a base path by running a command
similar to the following:

```sh
// Create a folder to use as the db base path
$ mkdir my-chain-state

// Use of that folder to store the chain state
$ ./target/release/node-template --dev --base-path ./my-chain-state/

// Check the folder structure created inside the base path after running the chain
$ ls ./my-chain-state
chains
$ ls ./my-chain-state/chains/
dev
$ ls ./my-chain-state/chains/dev
db keystore network
```

## Step 1: Include the Treasury Pallet

Ensure the `pallet-treasury` is included in your project's `Cargo.toml`.

```toml
[dependencies.pallet-treasury]
default-features = false
version = '3.0.0'  # Ensure the version matches your Substrate setup

```


Add the pallet to the `std` feature of your runtime:
```
std = [
    ...
    'pallet-treasury/std',
]

```


## Step 2: Configure the Treasury Pallet

Import the Pallet
```
use pallet_treasury;

```

Add to your runtime's c`onstruct_runtime!` macro:
```
Treasury: pallet_treasury::{Pallet, Call, Storage, Config, Event<T>},

```


Set up the necessary parameters using the `parameter_types!` macro:

```
parameter_types! {
    pub const ProposalBond: Permill = Permill::from_percent(5);
    pub const ProposalBondMinimum: Balance = 1 * DOLLARS;
    pub const SpendPeriod: BlockNumber = 1 * DAYS;
    pub const Burn: Permill = Permill::from_percent(0); // No funds burning
}

impl pallet_treasury::Config for Runtime {
    type Currency = Balances;
    type ApproveOrigin = EnsureRoot<AccountId>;
    type RejectOrigin = EnsureRoot<AccountId>;
    type RuntimeEvent = RuntimeEvent;
    type ProposalBond = ProposalBond;
    type ProposalBondMinimum = ProposalBondMinimum;
    type SpendPeriod = SpendPeriod;
    type Burn = Burn;
    type BurnDestination = (); // Optional destination for burnt funds
    type WeightInfo = pallet_treasury::weights::SubstrateWeight<Runtime>;
    type SpendFunds = ();
}

```


## Step 3: Submitting Proposals

Include logic for submitting spending proposals through an extrinsic that interacts with the `propose_spend` function of the Treasury pallet.

## Step 4: Set Up Governance for Approvals
Configure a governance mechanism to manage the approval process of proposals:

```
type ApproveOrigin = EnsureRoot<AccountId>;
type RejectOrigin = EnsureRoot<AccountId>;

```
This README will help new contributors or team members understand the Treasury setup in your blockchain project.