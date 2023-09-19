# Tutorials

## About Ethereum merge
The Merge was the joining of the original execution layer of Ethereum (the Mainnet that has existed since genesis) with its new proof-of-stake consensus layer, the Beacon Chain. It eliminated the need for energy-intensive mining and instead enabled the network to be secured using staked ETH. It was a truly exciting step in realizing the Ethereum vision—more scalability, security, and sustainability.

![image](images/the_merge.png)

## Software needs to be installed

### Geth (Execution client)

Go-ethereum can be installed with the built images which are downable here: https://geth.ethereum.org/downloads

Detailed installation instructions can be found here: https://geth.ethereum.org/docs/getting-started/installing-geth

A short Intro to Geth: https://geth.ethereum.org/docs/getting-started

### Consensus clients

Geth is an execution client. Historically, an execution client alone was enough to run a full Ethereum node. However, since Ethereum swapped from proof-of-work (PoW) to proof-of-stake (PoS) based consensus, Geth needs to be coupled to another piece of software called a "consensus client".

Intro link to consensus clients: https://geth.ethereum.org/docs/getting-started/consensus-clients

There are currently five consensus clients that can be run alongside Geth. These are:

- [Lighthouse](https://lighthouse-book.sigmaprime.io/): written in Rust
- [Nimbus](https://nimbus.team/): written in Nim
- [Prysm](https://docs.prylabs.network/docs/getting-started): written in Go
- [Teku](https://consensys.io/teku/): written in Java
- [Lodestar](https://lodestar.chainsafe.io/): written in Typescript

In this tutorial we will work with **Lighthouse** consensus client using the source code: https://lighthouse-book.sigmaprime.io/installation-source.html.

### Clef (Account manager)

Intro to Clef: https://geth.ethereum.org/docs/tools/clef/introduction

Geth uses an external signer called Clef to manage accounts. This is a standalone piece of software that runs independently of - but connects to - a Geth instance. Clef handles account creation, key management and signing transactions/data.

Clef comes bundled with Geth and can be built along with Geth and the other bundled tools using:

```shell
make all
```

However, Clef is not bound to Geth and can be built on its own using:

```shell
make clef
```

Once built, Clef must be initialized. This includes storing some data, some of which is sensitive (such as passwords, account data, signing rules etc). Initializing Clef takes that data and encrypts it using a user-defined password.

```shell
clef init
```

## Getting started

> For further info, visit the below links:
> 1. Geth source: https://geth.ethereum.org/docs/getting-started
> 2. Lighthouse sources: https://lighthouse-book.sigmaprime.io/run_a_node.html

After successful installations, we can do as follows in order for getting started:

### step 0: generating JWT

create a directory as a data/key store
```shell
mkdir geth-tutorial
```

And then: 

```shell
openssl rand -hex 32 | tr -d "\n" | sudo tee geth-tutorial/jwtsecret
```

### step 1: Generating accounts
There are several methods for generating accounts in Geth. This tutorial demonstrates how to generate accounts using Clef, as this is considered best practice, largely because it decouples the users' key management from Geth, making it more modular and flexible.

An account is a pair of keys (public and private). Clef needs to know where to save these keys to so that they can be retrieved later. This information is passed to Clef as an argument. This is achieved using the following command:

```shell
clef newaccount --keystore geth-tutorial/keystore
```

### Step 2: Start Clef

The previous commands used Clef's newaccount function to add new key pairs to the keystore. Clef uses the private key(s) saved in the keystore to sign transactions. In order to do this, Clef needs to be started and left running while Geth is running simultaneously, so that the two programs can communicate between one another.

>Note: Ethereum mainnet has chain ID 1. In this tutorial Chain ID 11155111 is used which is that of the Sepolia testnet.

The following command starts Clef on Sepolia:

```shell
clef --keystore geth-tutorial/keystore --configdir geth-tutorial/clef --chainid 11155111
```

### Step 3.1: Start Geth

The final argument passed to Geth is the --http flag.This enables the http-rpc server that allows external programs to interact with Geth by sending it http requests. By default the http server is only exposed locally using port 8545: localhost:8545. It is also necessary to authorize some traffic for the consensus client which is done using --authrpc and also to set up a JWT secret token in a known location, using --jwt-secret.

The following command should be run in a new terminal, separate to the one running Clef:

```shell
geth --sepolia --datadir geth-tutorial --authrpc.addr localhost --authrpc.port 8551 --authrpc.vhosts localhost --authrpc.jwtsecret geth-tutorial/jwtsecret --http --http.api eth,net --signer=geth-tutorial/clef/clef.ipc --http
```

### Step 3.2: Start Lighthouse

In this step, we will set up a beacon node. Use the following command to start a beacon node that connects to the execution node:

```shell
lighthouse bn \
  --network sepolia \
  --execution-endpoint http://localhost:8551 \
  --execution-jwt geth-tutorial/jwtsecret \
  --checkpoint-sync-url https://sepolia.beaconstate.info \
  --http
```

> Note:
> --network flag, which selects a network:
> - lighthouse (no flag): Mainnet.
> - lighthouse --network mainnet: Mainnet.
> - lighthouse --network goerli: Goerli (testnet).
> - lighthouse --network sepolia: Sepolia (testnet).
> - lighthouse --network chiado: Chiado (testnet).
> - lighthouse --network gnosis: Gnosis chain.

<mark>**Do we have any option for a private network?**</mark>

> Note:
> Public endpoints: https://eth-clients.github.io/checkpoint-sync-endpoints/

### Step 4: Get Testnet Ether

https://sepoliafaucet.com/

