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

I used this key to generate `keystore.json`:

## 2.1 Generating `keystore` file in `deployer`

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ starkli signer keystore from-key ./keystore.json
Enter private key: 
Enter password: 
Created new encrypted keystore file: /home/hp/.starkli-wallets/deployer/keystore.json
Public key: 0x079...
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
    "implementation": "0x5...",
    "multisig": {
      "status": "off"
    },
    "signers": [
      {
        "type": "stark",
        "public_key": "0x799..."
      }
    ]
  },
  "deployment": {
    "status": "deployed",
    "class_hash": "0x31...",
    "address": "0x1a8c..."
  }
}
```

## 2.3 Creating `envars.sh` environment variables file

Here's my `envars.sh`:

```sh
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/deployer/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/deployer/account.json
export STARKNET_RPC=https://starknet-goerli.infura.io/v3/a6e435c..

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

## 3.1 Create `keystore.json` for `custom`

```bash
hp@Cyndie:~/.starkli-wallets/custom$ starkli signer keystore new ./keystore.json
Enter password: 
Created new encrypted keystore file: /home/hp/.starkli-wallets/custom/keystore.json
Public key: 0x6...
hp@Cyndie:~/.starkli-wallets/custom$ 
```

## 3.2 Create `envars.sh` to store account contract's environment variables

First store the signer's environment variables:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ cat envars.sh 
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/custom/keystore.json
```

Activate it:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ source envars.sh 
hp@Cyndie:~/.starkli-wallets/custom$ 
```

## 3.3 Create `account.json` for account contract

We'll use a Starkli command normally reserved for an OZ account, even though the account contract is made from scratch.

This is because our account contract validates signatures the same as OpenZeppelin

If your account contract does it differently you won't use Starkli to deploy it.

```bash
hp@Cyndie:~/.starkli-wallets/custom$ starkli account oz init ./account.json
Enter keystore password: 
Created new account config file: /home/hp/.starkli-wallets/custom/account.json

Once deployed, this account will be available at:
    0x06f..

Deploy this account by running:
    starkli account deploy ./account.json
hp@Cyndie:~/.starkli-wallets/custom$ 
```

Here's the generated `account.json`:

```json
{
  "version": 1,
  "variant": {
    "type": "open_zeppelin",
    "version": 1,
    "public_key": "0x6ee..",
    "legacy": false
  },
  "deployment": {
    "status": "undeployed",
    "class_hash": "0x4c6..",
    "salt": "0x48911..."
  }
}
```

## 3.4 Calculate the correct  account contract's Class Hash

I compiled the smart contract first:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa/src$ ls
lib.cairo
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa/src$ scarb build
   Compiling aa v0.1.0 (/home/hp/Desktop/account_abstraction__starknet/aa/Scarb.toml)
    Finished release target(s) in 2 seconds
```

I then derived the Class Hash using Starkli:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ starkli class-hash ./target/dev/aa_Account.sierra.json
0x03480253c19b447b1d7e7a6422acf80b73866522de03126fa55796a712d9f092
```

I modified the account contract's `account.json` to hold the generated class hash:

```json
{
  "version": 1,
  "variant": {
    "type": "open_zeppelin",
    "version": 1,
    "public_key": "0x6ee...",
    "legacy": false
  },
  "deployment": {
    "status": "undeployed",
    "class_hash": "0x034...",
    "salt": "0x4891..."
  }
}
```

I added `account.json` to `envars.sh` and activated it:

```sh
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/custom/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/custom/account.json

```

```bash
hp@Cyndie:~/.starkli-wallets/custom$ source envars.sh 
hp@Cyndie:~/.starkli-wallets/custom$ 
```

## 4. Configure the RPC

The next step is declaring the account contract, and we'll do that through an RPC provider: Infura, 
or a Starknet node;

I got the API key from Infura, and added it to `envars.sh`, similar to `deployer` environment variable:

```sh
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/custom/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/custom/account.json
export STARKNET_RPC=https://starknet-goerli.infura.io/v3/a6e435....
```

## 5. Declaring the account contract

I first activated the `deployer` environment in the `aa` directory:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ source ~/.starkli-wallets/deployer/envars.sh
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ 
```

I then declared the account contract:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ starkli declare target/dev/aa_Account.sierra.json
Enter keystore password: 
Not declaring class as it's already declared. Class hash:
0x03480253c19b447b1d7e7a6422acf80b73866522de03126fa55796a712d9f092
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ 
```

## 6. Deploying the account contract

I reactivated the `custom` environment variables in the `aa` directory for account deployment:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ source ~/.starkli-wallets/custom/envars.sh
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ 
```

I then started the deployment procedure:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ starkli account deploy ~/.starkli-wallets/custom/account.json
Enter keystore password: 
The estimated account deployment fee is 0.000004330000038970 ETH. However, to avoid failure, fund at least:
    0.000006495000058455 ETH
to the following address:
    0x07..
Press [ENTER] once you've funded the address.
Account deployment transaction: 0x0434...
Waiting for transaction 0x0434... to confirm. If this process is interrupted, you will need to run `starkli account fetch` to update the account file.
Transaction not confirmed yet...
Transaction not confirmed yet...
Transaction 0x0434... confirmed
```

## N.B

I used to get this error while deploying my contract using Alchemy as my RPC endpoint:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ starkli account deploy ~/.starkli-wallets/custom/account.json
Enter keystore password: 
The estimated account deployment fee is 0.000004330000051960 ETH. However, to avoid failure, fund at least:
    0.000006495000077940 ETH
to the following address:
    0x00...
Press [ENTER] once you've funded the address.
Account deployment transaction: 0x05...
Waiting for transaction 0x058... to confirm. If this process is interrupted, you will need to run `starkli account fetch` to update the account file.
Transaction not confirmed yet...
Transaction not confirmed yet...
Error: data did not match any variant of untagged enum JsonRpcResponse

```

Apparently the current Starkli version (v0.1.19) supports only JSON-RPC spec version (0.4.0), and Alchemy is operating on (0.3.0)

https://github.com/xJonathanLEI/starkli/issues/44#issuecomment-1803358124



