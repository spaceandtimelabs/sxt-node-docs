# SXT Chain Testnet Validator Setup Instructions

## Introduction
### Launching SXT Chain – Background and Timelines
#### Introduction to SXT Chain
* **Overview**: Space and Time (SXT) scales zero-knowledge (ZK) proofs on a decentralized data warehouse to deliver trustless data processing to smart contracts. You can use SXT to join comprehensive blockchain data we've indexed from major chains with apps’ data or other offchain datasets. Proof of SQL is SXT's sub-second ZK coprocessor, which allows smart contracts to ask complicated questions about activity on its own chain or other chains and get back a ZK-proven answer in the next block. SXT enables a new generation of smart contracts that can transact in real time based on both onchain data (like txns, blocks, smart contract events, storage slot changes, etc.) and data from apps, ultimately delivering a more robust onchain economy and more sophisticated onchain applications.
* **Validator activities**: The SXT Chain is designed to witness and validate indexed data submitted by indexer nodes. While the current testnet phase only involves third-party validators (and not prover nodes quite yet), the SXT Chain ensures the integrity of data through cryptographic commitments—one special hash for each indexed table. As rows of data are inserted on the SXT Chain, the validators update these special hashes (commitments) and add them to each block. Ultimately, this proves/ensures that each table managed by the chain is cryptographically tamperproof.
* **Data Integrity**: As the SXT Chain indexes data, it generates cryptographic hashes (commitments) that guarantee the underlying data tables remain untampered. These commitments are essential for later phases, where our SXT Chain prover nodes will employ our Proof of SQL protocol to validate the integrity of the indexed data during ZK-proven SQL execution for client queries.

> [!IMPORTANT]
> If you were a validator in the original SXT Testnet, you will need to start a new validator from scratch using the instructions below.

## Noteworthy Mentions
> [!IMPORTANT]
> Traditionally Substrate networks rely on Substrate-compatible keys to register node operators and manage staking.
> The new Space and Time Testnet does not use Substrate keys. Stakers and Node operators will only use Ethereum keys in an Ethereum wallet.
> Stakers and node operators will use their Ethereum keys to interact with Space and Time's Staking and Messaging contracts. Transactions that take place in these contracts will then be reflected on-chain.

## I. Prerequisites

#### Incentives and Expectations
* **Participation Requirements**: Validators are expected to actively participate, with a response SLA of 48 hours for notifications around actions needed from validator operators, chain upgrades, changes to testing schedule, etc.
* **Incentive Structure**: We may choose to provide rewards to eligible Testnet participants for completing certain activities, including running nodes, performing other critical services related to the SxT Chain, and meeting certain performance criteria or other requirements. Additional details will be provided to you from time to time via Testnet-related websites and developer documentation.

**Discord information**: We have set up a Testnet Nodes channel in the SXT [Discord](https://discord.com/invite/spaceandtimeDB) to manage all communications and Q&A with node operators during testnet.

![image1](./assets/image1.png)

![image2](./assets/image2.png)


You will only be able to access this channel list if you hold the “Testnet Node Operator” role in the SXT Discord. Please share any team members’ Discord usernames with us directly via email at testnet@spaceandtime.io and we will assign this role to their profile(s).

### 1.1. System Specifications
The minimum system requirements for running a SXT validator node are shown in the table below:

| Key              | Value           |
|------------------|-----------------|
| CPU cores        | 16              |
| CPU Architecture | amd64           |
| Memory (GiB)     | 64              |
| Storage (GiB)    | 512             |
| Storage Type     | SSD             |
| OS               | Linux           |
| Network Speed    | 500Mbps up/down |

On Azure cloud, this is equivalent to SKU `Standard_D8as_v5` with storage SKU of `PremiumV2_SSD`.

### 1.2. Downloads
#### 1.2.1. Docker Image
Assuming Docker Desktop is installed and working on your computer. The SXT Node Docker image can be downloaded with `docker pull` command.

```bash
docker pull ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0
docker images --digests ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0
```

When each new docker image is released we will also be sharing the full `sha256` hash of the image. Please confirm that hash against the image pulled down by docker with an extra docker `images` argument `--digests` to make sure that you are pulling the right one.

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.107.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

#### 1.2.2. Testnet Chainspecs
SXT testnet chainspecs are part of the docker images mentioned in [section 1.2.1](#121-docker-image). To inspect the chainspecs that come with the docker image, please run the following:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  --entrypoint=bash ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0 \
  -c "cat /opt/chainspecs/testnet-spec.json"
```

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.107.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

### 1.3. Testnet Bootnodes
Bootnodes on SXT networks are trusted peers on the network that a new node will first connect to and find more peers to download blocks from. The three bootnodes listed below are hosted by Space and Time:

```
/dns/new-validator0.testnet.sxt.network/tcp/30333/p2p/12D3KooWDV5kmYUR5nxruFBfdGX2ZMR43iSe3SfmopZ3sLBFvZzc
/dns/new-validator1.testnet.sxt.network/tcp/30333/p2p/12D3KooWGAQAg7iZgyn8wnnT8nkDo9NVAPbfubpMgL1mYTRNgjdD
/dns/new-validator2.testnet.sxt.network/tcp/30333/p2p/12D3KooWLLf8tW3PPbj9MCda9rfypNN5xyZRi1bKoLj8s9UkeJDZ
```
### 1.4. Node Keys
Because the SxT Chain relies on EVM contracts for staking, node operators will need an Ethereum wallet (or Sepolia for Testnet) to interact with the staking contracts. The wallet you're using should have at least 0.05 ETH for transaction fees on the networks.

> [!IMPORTANT]
> Traditionally Substrate networks rely on Substrate-compatible keys to register node operators and manage staking.
> The new Space and Time Testnet does not use Substrate keys. Stakers and Node operators will only use Ethereum keys in an Ethereum wallet.
> Stakers and node operators will use their Ethereum keys to interact with Space and Time's Staking and Messaging contracts. Transactions that take place in these contracts will then be reflected on-chain.

A validator node key is used to create a node's peer id in order to uniquely identify the node over the p2p network. We first create a folder where we want to store the node-key, then mount the folder into docker image and run the key generating command:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  -v sxt-node-key:/data \
  --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0 \
  key generate-node-key --chain /opt/chainspecs/testnet-spec.json --file /data/subkey.key
```

The generated key should now be in a file called `subkey.key` in the sxt-node-key volume. Note that from the command line output it should also show you the peer id of the node.

#### Validator Keys Permission Workaround
A workaround to fix a permission issue in the sxt-validator-key volume needs to be applied as well. This will be fixed in a future release:
```bash
docker run -it --rm \
  --platform linux/amd64 \
  -v sxt-validator-key:/key \
  --user root \
  --entrypoint=chown \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0 \
  -R sxtuser:sxtuser /key
```

## SXT Chain Testnet: NPoS Staking Instructions

> [!NOTE]
> Please see the FAQ section at the end of this document if you have additional questions about onboarding as a validator

---

### Validators
Validators are the ones running the hardware that is creating blocks and participating in Consensus. Validators have their own stake in addition to anyone nominating them.

### Nominators
Nominators can allocate their stake towards an existing validator and participate in a portion of staking rewards without having to run their own hardware. In the event that a Validator is slashed for acting badly, the nominators will also be slashed. Nominators can nominate multiple validators.

### Eras
Every Era the elected set changes based on the distribution of stake from validators and nominators. Eras rotate every 24 hours.

### Epochs
Every Epoch new block slots are assigned to the previously elected validator set.

### Elections
Elections take place in the last block of the next-to-last Epoch. For example, SXT Chain has 24 Hour Eras consisting of six 4 hour long Epochs.
At the last block of Epoch 5 in each era, the election will take place and keys for the new validators will be queued to become active at the start of the next Era.


### Testnet Contract Addresses (Sepolia):

- **Testnet Staking Contract**
  [0xca755ce69181d2d33097a24ce5ddc030a0b87f2c](https://sepolia.etherscan.io/address/0xca755ce69181d2d33097a24ce5ddc030a0b87f2c#writeContract)


- **Testnet Token Contract**
  [0xdB5FFD69Fa8022e20cf7d66C36c961a2949f4Da0](https://sepolia.etherscan.io/token/0xdb5ffd69fa8022e20cf7d66c36c961a2949f4da0#writeContract)

- **Testnet SessionKey Registration Contract**
  [0x82840556980bfbCc08e3e7c61AA44E1a4EAb5471](https://sepolia.etherscan.io/address/0x82840556980bfbcc08e3e7c61aa44e1a4eab5471#writeContract)

---

### Pre-Requisites

- Ethereum wallet with Sepolia ETH
- A minimum balance of **0.05 Sepolia ETH**
- Synced **SXT Chain validator node**
---

## Steps

### Step 1: Request Testnet Tokens
- You can use the faucet to receive Testnet SXT periodically: https://sxtchaintestnetfaucet.vercel.app/

---

### Step 2: Approve Token Spend
Send a transaction to the token contract to approve the staking contract to spend your test tokens:
- [0xdB5FFD69Fa8022e20cf7d66C36c961a2949f4Da0](https://sepolia.etherscan.io/address/0xdB5FFD69Fa8022e20cf7d66C36c961a2949f4Da0)
- Send an `approve` transaction with:
  - The **staking contract address** 0xca755ce69181d2d33097a24ce5ddc030a0b87f2c
  - The **token limit** to approve
  ![Etherscan Approval Transaction](./assets/approve.png)
---

### Step 3: Stake Tokens
Stake your desired amount using the **staking contract**. You must stake a minimum of 100 SXT or 100000000000000000000 units
- [0xca755ce69181d2d33097a24ce5ddc030a0b87f2c](https://sepolia.etherscan.io/address/0xca755ce69181d2d33097a24ce5ddc030a0b87f2c)
  ![Etherscan Stake Transaction](./assets/stake.png)
---

## Validators Only

### Register Your Session Keys

Use the message transaction to submit your session keys.

Call `rotateKeys()` RPC on your node:

```bash
curl -X POST http://localhost:9944 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "author_rotateKeys",
    "params": [],
    "id": 1
  }'
```

You’ll receive a response like:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x3084486e870e12fc551eacc173291f0d75ac5fed823aeb1e158bc98db215936202a555f88490d19f7fbacac7078fc87886084efd8227187a73ad05aee6da8ad38edd8739daa5689e9e118eb3be0330bbf80a30ad7639d4f0d70970dbccff9c4a"
}
```

- Copy the `result` hex string.
- Paste it into the `body` field of the **message transaction**.
- This also triggers `validate()` to activate your node.
  ![Etherscan Register Keys Transaction](./assets/message.png)

> [!IMPORTANT]
> You might see the following error due to file permissions. If that's the case, follow these [workaround instructions](#validator-keys-permission-workaround):
```
{
   "error" : {
      "code" : 1040,
      "message" : "Client error: Execution failed: Execution aborted due to trap: host code panicked while being called by the runtime: `sr25519_generate` failed: Other(\"Permission denied (os error 13)\")\nWASM backtrace:\nerror while executing at wasm backtrace:\n    0: 0x1a38d8 - sxt_runtime.wasm!sp_io::crypto::extern_host_function_impls::sr25519_generate::hd241fcbcc7fb892e\n    1: 0x58547a - sxt_runtime.wasm!sxt_runtime::opaque::SessionKeys::generate::hfa56532150c4f3a1\n    2: 0x4e21b0 - sxt_runtime.wasm!SessionKeys_generate_session_keys"
   },
   "id" : 1,
   "jsonrpc" : "2.0"
}
```

---

## How to Nominate (Optional)

This is an optional step; in order to nominate someone, they must be an active validator. You can nominate validators by submitting their **hexadecimal** form of the wallet address as it appears on Substrate to the staking contract. This can be found from the validator list and then converted from SS58 to Hexadecimal.

In order to generate the hexadecimal value from the SS58 value, one can run the following commands:

```bash
SS58_KEY=<The SS58 public wallet address of the validator you want to nominate>
docker run -it --rm \
  --platform linux/amd64 \
  -v sxt-node-key:/data \
  --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0 \
  key inspect --chain /opt/chainspecs/testnet-spec.json $SS58_KEY 
```

The SS58_KEY can be obtained from the address list of validators in the [Staking Dashboard](https://polkadot.js.org/apps/?rpc=wss://new-rpc.testnet.sxt.network/#/staking)

**You MUST use Hex format. Do NOT use SS58 format**:

❌ Invalid (SS58):
`5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY`

✅ Valid (Hex):
`0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d`

You can enter **multiple nominations** like this:

```js
[0x1234, 0x3456, 0x5678]
```
  ![Etherscan Nominate Transaction](./assets/nominate.png)
---

## II. Validator Setup Using Docker

Here we assume the setup uses the following volumes: `sxt-testnet-data` is the block storage volume and the volume where the generated node key is stored is `sxt-node-key`.

### 2.1. Docker Run
```bash
docker run -d --restart always \
  --platform linux/amd64 \
  -v sxt-testnet-data:/data \
  -v sxt-validator-key:/key \
  -v sxt-node-key:/node-key \
  -p 30333:30333/tcp \
  -p 9615:9615/tcp \
  -p 9944:9944/tcp \
  --env HYPER_KZG_PUBLIC_SETUP_DIRECTORY=/data \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0 \
  --base-path /data \
  --prometheus-port 9615 \
  --prometheus-external \
  --pool-limit 10240 \
  --pool-kbytes 1024000 \
  --chain /opt/chainspecs/testnet-spec.json \
  --keystore-path /key \
  --node-key-file /node-key/subkey.key \
  --bootnodes "/dns/new-validator0.testnet.sxt.network/tcp/30333/p2p/12D3KooWDV5kmYUR5nxruFBfdGX2ZMR43iSe3SfmopZ3sLBFvZzc" \
  --bootnodes "/dns/new-validator1.testnet.sxt.network/tcp/30333/p2p/12D3KooWGAQAg7iZgyn8wnnT8nkDo9NVAPbfubpMgL1mYTRNgjdD" \
  --bootnodes "/dns/new-validator2.testnet.sxt.network/tcp/30333/p2p/12D3KooWLLf8tW3PPbj9MCda9rfypNN5xyZRi1bKoLj8s9UkeJDZ" \
  --validator \
  --port 30333 \
  --log info \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 5' \
  --no-private-ipv4 \
  --name ValidatorName \
  # only do the following if local RPC is desired
  --rpc-external \
  --rpc-cors all \
  --rpc-port 9944
```

### 2.2. Docker Compose
Prepare a `docker-compose.yaml` file as follows:

```yaml
---
name: 'sxt-testnet-node'

services:
  sxt-testnet:
    platform: linux/amd64
    restart: unless-stopped
    image: ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.107.0
    ports:
      - '9615:9615' # metrics
      - '9944:9944' # rpc
      - '30333:30333' # p2p
    volumes:
      - sxt-testnet-data:/data
      - sxt-validator-key:/key
      - sxt-node-key:/node-key
    pid: host
    environment:
      HYPER_KZG_PUBLIC_SETUP_DIRECTORY: /data
    command: >
      --base-path /data
      --prometheus-port 9615
      --prometheus-external
      --pool-limit 10240
      --pool-kbytes 1024000
      --chain /opt/chainspecs/testnet-spec.json
      --keystore-path /key
      --node-key-file /node-key/subkey.key
      --bootnodes "/dns/new-validator0.testnet.sxt.network/tcp/30333/p2p/12D3KooWDV5kmYUR5nxruFBfdGX2ZMR43iSe3SfmopZ3sLBFvZzc"
      --bootnodes "/dns/new-validator1.testnet.sxt.network/tcp/30333/p2p/12D3KooWGAQAg7iZgyn8wnnT8nkDo9NVAPbfubpMgL1mYTRNgjdD"
      --bootnodes "/dns/new-validator2.testnet.sxt.network/tcp/30333/p2p/12D3KooWLLf8tW3PPbj9MCda9rfypNN5xyZRi1bKoLj8s9UkeJDZ"
      --validator
      --port 30333
      --log info
      --telemetry-url 'wss://telemetry.polkadot.io/submit/ 5'
      --no-private-ipv4
      --name ValidatorName
      # only do the following if local RPC is desired
      --rpc-external
      --rpc-cors all
      --rpc-port 9944

volumes:
  sxt-testnet-data:
    external: true
  sxt-validator-key:
    external: true
  sxt-node-key:
    external: true
```

and then start the sxt-testnet-node with command below:

```bash
docker compose -f ./docker-compose.yaml up -d
```

# FAQ (More Coming Soon)

## Do I need to nominate my own validator?
No you do not need to nominate yourself if you are a validator. Your validator node has its own stake tied to your account.

## Where do I get the address if I do want to nominate someone else's validator?
You can find the SS58 address of all available validators in the [Polkadot Explorer](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fnew-rpc.testnet.sxt.network%2F#/explorer)
