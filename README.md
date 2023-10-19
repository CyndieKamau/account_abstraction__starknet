# Account Abstraction in Starknet

Account Abstraction is simply creating smart contract account wallets suitable to 
the needs of a user.

To understand Account Abstraction let's first quickly go through current
Ethereum's accounts model.


There are two types of accounts on Ethereum;

* Externally Owned Accounts (EOAs)

* Contract Accounts (CA)

Learn more about Contract Accounts [here](https://ethereum.org/en/developers/docs/accounts/)

## Externally Owned Accounts

EOAs are accounts owned by something away from the blockchain, i.e the users.

EOAs have three properties;

* An address to uniquely identify the account on the network

* A nonce to ensure every tx is unique

* Balance representing amount of ETH available to the account

To change the state of the blockchain, thus the account, transactions are performed.
The trigger must come externally and not in the blockchain.
So in Ethereum, EOAs trigger txs, which in turn modify the blockchain's state.

In summary, when a tx is executed by the Ethereum Virtual Machine (EVM), the 
first account to be touched is an EOA, and the corresponding account pays
the miner's fee for the tx to be executed.


# Creating a Simple account in Starknet

## Project Setup

Ensure you have [Scarb installed](https://docs.swmansion.com/scarb/download.html)

```bash
$ scarb --version
scarb 0.7.0 (58cc88efb 2023-08-23)
cairo: 2.2.0 (https://crates.io/crates/cairo-lang-compiler/2.2.0)
sierra: 1.3.0

``` 

Create a new project using Scarb;

```bash
$ scarb new simple_account
Created `simple_account` package.
```

Here's how the initial project setup looks like;

```bash
$ tree simple_account/
simple_account/
├── Scarb.toml
└── src
    └── lib.cairo

1 directory, 2 files

```
By default, Scarb initializes the project for Vanilla Cairo
instead of Starknet smart contracts.

We'll configure `Scarb.toml` file to handle Starknet smart contracts;

```sh
[package]
name = "simple_account"
version = "0.1.0"

# See more keys and their definitions at https://docs.swmansion.com/scarb/docs/reference/manifest.html

[dependencies]
starknet = "2.2.0"

[[target.starknet-contract]]

```





