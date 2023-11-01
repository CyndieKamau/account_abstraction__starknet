# This is an update of smart contract account deployment using Starkli v0.1.19

The previous tutorial was made using `0.1.17`, as of 1st Nov 2023 the current Starkli version is `0.1.19`.

## 1. Updating Starkli
The first thing I've done is to upgrade Starkli using Starkliup:

```bash
hp@Cyndie:~$ starkliup
Installing the latest version of starkli...
Fetching the latest release tag from GitHub...
Latest version found: v0.1.19
Detected host triple: x86_64-unknown-linux-gnu
Downloading release file from GitHub...
################################################################################################################################ 100.0%
Successfully installed starkli 

Generating shell completion files...
- Bash ... Done
- Zsh ... Done
Note that shell completions might not work until you start a new session.

Installation successfully completed.

```

Restart the terminal, then check on the Starkli version:

```bash
hp@Cyndie:~$ starkli --version
0.1.19 (3fd85cf)
``` 
## 2. Configure `deployer` directory in `./starkli-wallets`

I removed the previous `~/.starkli-wallets` directory to create a new one:

```bash
hp@Cyndie:~$ rm -rf ~/.starkli-wallets/
hp@Cyndie:~$ mkdir  ~/.starkli-wallets/
```

Created the `deployer` directory:

```bash
hp@Cyndie:~/.starkli-wallets$ mkdir deployer
hp@Cyndie:~/.starkli-wallets$ 
```

I then created a new smart wallet, funded it, then copied its private key.
Wallet has no resources, so I'll just expose the private key:
NOTE: Don't use a smart wallet having real funds!!!

```bash
0x039d3113b3fe945187b44eda2e511b27e53cc011da3118d1ad601cc3f250648e
```
I used this key to generate `keystore.json`:

## 2.1 Generating `keystore` file in `deployer`

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ starkli signer keystore from-key ./keystore.json
Enter private key: 
Enter password: 
Created new encrypted keystore file: /home/hp/.starkli-wallets/deployer/keystore.json
Public key: 0x079947340c32624fb60a83c8ff8912cd1ec00e6cd10c6ab228e17b4b4a869fa3
hp@Cyndie:~/.starkli-wallets/deployer$ 
```

## 2.2 Generating `account.json` file in `deployer`

In my first tutorial I was tring to fetch the account descriptor file for my smart wallet, but forgot to add
`--output` flag:

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ starkli account fetch --output ./account.json 0x01a8cfb468cd4bda3a71e05c517eb1e217467d9f11babc7637a8f0dbe92276f0 
WARNING: no valid provider option found. Falling back to using the sequencer gateway for the goerli-1 network. Doing this is discouraged. See https://book.starkli.rs/providers for more details.
Account contract type identified as: Braavos
Description: Braavos official proxy account
Downloaded new account config file: /home/hp/.starkli-wallets/deployer/account.json
hp@Cyndie:~/.starkli-wallets/deployer$ ls
account.json  keystore.json
```

Here's how the generated `account.json` file looks like:

```json
{
  "version": 1,
  "variant": {
    "type": "braavos",
    "version": 1,
    "implementation": "0x5dec330eebf36c8672b60db4a718d44762d3ae6d1333e553197acb47ee5a062",
    "multisig": {
      "status": "off"
    },
    "signers": [
      {
        "type": "stark",
        "public_key": "0x79947340c32624fb60a83c8ff8912cd1ec00e6cd10c6ab228e17b4b4a869fa3"
      }
    ]
  },
  "deployment": {
    "status": "deployed",
    "class_hash": "0x3131fa018d520a037686ce3efddeab8f28895662f019ca3ca18a626650f7d1e",
    "address": "0x1a8cfb468cd4bda3a71e05c517eb1e217467d9f11babc7637a8f0dbe92276f0"
  }
}
```

## 2.3 Creating `envars.sh` environment variables file

Here's my `envars.sh`:

```sh
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/deployer/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/deployer/account.json
export STARKNET_RPC=https://starknet-goerli.g.alchemy.com/v2/4VUpCv...

```

I then activated the environments:

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ source envars.sh 
hp@Cyndie:~/.starkli-wallets/deployer$ 
```
## 3. Configuration for the account contract

I created a new folder called `custom`:

```bash
hp@Cyndie:~/.starkli-wallets$ mkdir custom
hp@Cyndie:~/.starkli-wallets$ 

```

