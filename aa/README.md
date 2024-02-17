# Account Deployment using Starkli

Starkli is a Starknet CLI tool built using Rust.

## Starkli Installation

Although the tool is built in Rust you don't need rust in your system to run it.
Starkli provides a library manager called Starkliup, similar to rustup.

To install Starkliup:

```bash
 $ curl https://get.starkli.sh | sh
>>>
...
Run '. /home/parallels/.starkli/env' or start a new terminal session to use starkliup.
...
```
Either restart your shell or execute the suggested command in your terminal.

With Starkliup installed use it to install Starkli

```bash
$ starkliup
>>>

Installing the latest version of starkli...
...
Note that shell completions might not work until you start a new session.
Installation successfully completed.
``` 

Restart your terminal.

Check if Starkli is installed:

```bash
hp@Cyndie:~/.starkli-wallets$ starkli --version
0.1.17 (2e1d7a8)
```

## Creating a wallet for the CLI

Declaring, deploying and invoking a smart contract in the CLI requires paying for gas fees,
which needs a wallet.

Create a folder which will hold all the wallets you might create in future.

```bash
$ mkdir ~/.starkli-wallets
```

For each smart wallet, there will be two files, a **signer** and an **account descriptor**

Create a folder called `deployer`, which we will use later on while declaring our account contract:

```bash
$ mkdir ~/.starkli-wallets/deployer
```

We'll integrate Braavos smart wallet to configure our `deployer` folder, which will be used when declaring and deploying
our account contract later on.

## Configuring Braavos as deployer in Starkli

The first step is creating a new smart wallet using Braavos browser extension, which we will use only
for Declaring and Deploying smart contracts.

Don't reuse the same wallet where you hold tokens or NFTs as we'll be exposing the private key to Starkli.

After creating a new smart wallet and funding it using test ETH [here](https://faucet.goerli.starknet.io/),
it's time to export the wallet's private key, to be used in Starkli.


In your Braavos smart wallet, copy the Private key.

## Creating a Signer

We'll use Starkli to create a `keystore.json` file inside the `deployer` directory, where we'll use Braavos's smart wallet
private key which you have copied;

```bash
hp@Cyndie:~/.starkli-wallets$ cd deployer/
hp@Cyndie:~/.starkli-wallets/deployer$ starkli signer keystore from-key ./keystore.json
Enter private key: 
Enter password: 
Created new encrypted keystore file: /home/hp/.starkli-wallets/deployer/keystore.json
Public key: 0x0519c51e245a4a63fd7773f8658c12277860e77cd30a8727fc2a22266c9e1c4c

```
The `private key` is the one copied from Braavos.

The `password` is a password you'll create to encrypt the file.

We'll then create an Account descriptor file to inform Starkli how our smart wallet created by Braavos 
will expect a user to sign transactions.

## Creating an Account

Create an `account.json` file in the `deployer` folder, where we'll add the details of our smart wallet.

```json
{
  "version": 1,
  "variant": {
  "type": "open_zeppelin",
  "version": 1,
    "public_key": "<SMART_WALLET_PUBLIC_KEY>"
  },
  "deployment": {
    "status": "deployed",
    "class_hash": "<SMART_WALLET_CLASS_HASH>",
    "address": "<SMART_WALLET_ADDRESS>"
  }
}
```


- `public_key`: Braavos smart wallet key found in the account settings, together with the private key

- `class_hash`: Class hash of your smart wallet address found in [Starkscan](https://testnet.starkscan.co/) block explorer

- `address`: Smart wallet address


```bash
hp@Cyndie:~/.starkli-wallets/deployer$ nano account.json
hp@Cyndie:~/.starkli-wallets/deployer$ cat account.json 
{
  "version": 1,
  "variant": {
  "type": "open_zeppelin",
  "version": 1,
    "public_key": "0x5..."
  },
  "deployment": {
    "status": "deployed",
    "class_hash": "0x03131....",
    "address": "0x00...."
  }
}

```

N.B I did try using `Starkli account fetch` command to automatically fetch details of our smart wallet:

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ starkli account fetch 0x00901aa3302c920e57f527dc767c4c46131cd90afc49e303580d2a8ea5b83f1c
NOTE: --output is not supplied and the account config won't be persisted.
WARNING: no valid provider option found. Falling back to using the sequencer gateway for the goerli-1 network. Doing this is discouraged. See https://book.starkli.rs/providers for more details.
Account contract type identified as: Braavos
Description: Braavos official proxy account

```
It didn't automatically generate for me the `account.json` file.


We'll create `envars.sh` to hold all the environment configurations for our `deployer` wallet.
There are 3 main variables added here:

- RPC Provider -> You can use Alchemy, Infura, or your own Starknet node (We'll do this also later on)

- Location of `keystore.json` for the signer

- Location of `account.json` account descriptor file

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ cat envars.sh 
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/deployer/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/deployer/account.json
export STARKNET_RPC=https://starknet-goerli.g.alchemy.com/v2/4VUp...

```
Hide the RPC URL to protect your API Key.

To activate the environment variables, use `source`:

```bash
hp@Cyndie:~/.starkli-wallets/deployer$ source envars.sh 
hp@Cyndie:~/.starkli-wallets/deployer$  

```

We'll use this when declaring our account contract.


## Configuration files for our account contract

When dealing with an account contract, Starkli will need two configuration files;

- `keystore.json` -> An encrypted file to store the associated private key

- `account.json` -> An unencrypted file describing its public attributes eg public key, class hash, address.

- `envars.sh` -> An optional file created for each account contract to source environment variables expected by Starkli

To deploy our account, create a new folder to hold the smart wallet:

```bash
hp@Cyndie:~/.starkli-wallets$ mkdir custom
```

We'll use Starkli to create `keystore.json` and `account.json`, then manually create `envars.sh`.

## Create a Keystore file

Starkli auto-generates a private key , store it in `keystore.json` file in our `custom` folder, and encrypt the 
file with a password.

```bash
hp@Cyndie:~/.starkli-wallets$ starkli signer keystore new ./custom/keystore.json
Enter password: 
Created new encrypted keystore file: /home/hp/.starkli-wallets/custom/keystore.json
Public key: 0x01...

```
The command `starkli signer keystore new ./custom/keystore.json` asks you to input a password to encrypt the file,
then it creates the `keystore.json` file.

In our `envars.sh` file, the location of the `keystore.json` file will be the first thing to add.

Create an `envars.sh` file, then add the location of `keystore.json`:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ nano envars.sh
hp@Cyndie:~/.starkli-wallets/custom$ cat envars.sh 
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/custom/keystore.json

```
Activate `envars.sh` using `source`:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ source envars.sh 
hp@Cyndie:~/.starkli-wallets/custom$ 

```

## Creating `account.json`

Our account contract validates signatures the same way as Open Zeppelin's default account contract.

Both account contracts expect a single signer, and use STARK-friendly ecdsa signature.

We'll use a Starkli command normally reserved for an OZ account, even though the account contract is made from scratch.

```bash
hp@Cyndie:~/.starkli-wallets/custom$ starkli account oz init ./account.json
Enter keystore password: 
Created new account config file: /home/hp/.starkli-wallets/custom/account.json

Once deployed, this account will be available at:
    0x020746ecae88e0eea0da05770eddee1165515180acb35efeb27d1daad0aed418

Deploy this account by running:
    starkli account deploy ./account.json

```

Note that our account contract is similar to OZ but not the same.
The account contract will have a different class hash than OZ's implementation.
The class hash will be used to determine the deployer address, so for now 
` 0x020746ecae88e0eea0da05770eddee1165515180acb35efeb27d1daad0aed418` is incorrect.


Let's check out the contents of `account.json`:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ cat account.json 
{
  "version": 1,
  "variant": {
    "type": "open_zeppelin",
    "version": 1,
    "public_key": "0x14...",
    "legacy": false
  },
  "deployment": {
    "status": "undeployed",
    "class_hash": "0x4c....",
    "salt": "0x36c..."
  }
}

```
Let's find the correct class hash.

## Calculating the Class Hash

We'll go to the `aa` folder containing our account contract, and compile it.

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa/src$ scarb build
   Compiling aa v0.1.0 (/home/hp/Desktop/account_abstraction__starknet/aa/Scarb.toml)
    Finished release target(s) in 2 seconds

```
The compiled version of our account contract now lives in `target/dev`.

We'll derive the class hash of our account contract using Starkli:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ starkli class-hash ./target/dev/aa_Account.sierra.json 
0x03480253c19b447b1d7e7a6422acf80b73866522de03126fa55796a712d9f092

```

We'll now modify `account.json` to hold the derived class hash:


```bash
hp@Cyndie:~/.starkli-wallets/custom$ nano account.json 
hp@Cyndie:~/.starkli-wallets/custom$ cat account.json 
{
  "version": 1,
  "variant": {
    "type": "open_zeppelin",
    "version": 1,
    "public_key": "0x14....",
    "legacy": false
  },
  "deployment": {
    "status": "undeployed",
    "class_hash": "0x0348..",
    "salt": "0x3.."
  }
}

```
Our `account.json` has the right info, we'll create an environment variable for it in `envars.sh`:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ cat envars.sh 
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/custom/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/custom/account.json

```
Activate the new environment variable:

```bash
hp@Cyndie:~/.starkli-wallets/custom$ source envars.sh 
hp@Cyndie:~/.starkli-wallets/custom$ 

```

## Configuring an RPC

The next step is to declare the contract, which means sending a transaction to Starknet.
You can only do so through an RPC provider. You can use [Alchemy](https://www.alchemy.com/) 
or [Infura](https://www.infura.io/).

I have used Alchemy. In the website, create a new app, configure it to Starknet-goerli (starknet testnet), then 
copy the API key in https format. That's the RPC's URL.

we'll add the URL to our `envars.sh` file as Starknet's RPC.

```bash
hp@Cyndie:~/.starkli-wallets/custom$ cat envars.sh 
#!/bin/bash
export STARKNET_KEYSTORE=~/.starkli-wallets/custom/keystore.json
export STARKNET_ACCOUNT=~/.starkli-wallets/custom/account.json
export STARKNET_RPC=https://starknet-goerli.g.alchemy.com/v2/4VUp....

```
For security purposes its advisable not to showcase your full URL to protect your API key.

## Declaring the Account Contract

To declare our account contract we'll need a different account contract that is already deployed
and funded to pay for the associated gas fees.

We'll configure Starkli to use [Braavos](https://braavos.app/) as the deployer 
(you can use [Argent X](https://www.argent.xyz/argent-x/) too).

Since we had already configured the `deployer` account, we can now declare our account contract.

We'll first activate the `deployer` environment in our `aa` directory containing the account contract:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ source ~/.starkli-wallets/deployer/envars.sh
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$
```

After that we'll declare the account contract:

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ starkli declare target/dev/aa_Account.sierra.json 
Enter keystore password: 
Sierra compiler version not specified. Attempting to automatically decide version to use...
Network detected: goerli-1. Using the default compiler version for this network: 2.1.0. Use the --compiler-version flag to choose a different version.
Declaring Cairo 1 class: 0x03480253c19b447b1d7e7a6422acf80b73866522de03126fa55796a712d9f092
Compiling Sierra class to CASM with compiler version 2.1.0...
CASM class hash: 0x05f58755772dba81b3ff0abda60d1079da08a5b1211c115117a6d59faef60947
Contract declaration transaction: 0x0699fd6d80d7733c7ceba4b3516eba0c6200f6155d3c565fe95bc9d1ee86afcc
Class hash declared:
0x03480253c19b447b1d7e7a6422acf80b73866522de03126fa55796a712d9f092
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ 

```

Note that the `keystore password` is the password you added when creating `keystore.json` for `deployer`.

Once the account is declared, we can now deploy the account.

First reactivate the environment variables for our `custom` account, to perform counterfactual deployment.

```bash
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ source ~/.starkli-wallets/custom/envars.sh
hp@Cyndie:~/Desktop/account_abstraction__starknet/aa$ 

```


