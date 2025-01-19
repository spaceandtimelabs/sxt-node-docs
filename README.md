# SXT Chain Testnet Validator Setup Instructions

## Introduction
### Launching SXT Chain – Background and Timelines
#### Introduction to SXT Chain
* **Overview**: Space and Time (SXT) scales zero-knowledge (ZK) proofs on a decentralized data warehouse to deliver trustless data processing to smart contracts. You can use SXT to join comprehensive blockchain data we've indexed from major chains with apps’ data or other offchain datasets. Proof of SQL is SXT's sub-second ZK coprocessor, which allows smart contracts to ask complicated questions about activity on its own chain or other chains and get back a ZK-proven answer in the next block. SXT enables a new generation of smart contracts that can transact in real time based on both onchain data (like txns, blocks, smart contract events, storage slot changes, etc.) and data from apps, ultimately delivering a more robust onchain economy and more sophisticated onchain applications.
* **Validator activities**: The SXT Chain is designed to witness and validate indexed data submitted by indexer nodes. While the current testnet phase only involves third-party validators (and not prover nodes or indexers quite yet), the SXT Chain ensures the integrity of data through cryptographic commitments—one special hash for each indexed table of Ethereum data. As rows of data are inserted on the SXT Chain, the validators update these special hashes (commitments) and add them to each block. Ultimately, this proves/ensures that each table managed by the chain is cryptographically tamperproof.
* **Data Integrity**: As the SXT Chain indexes Ethereum data, it generates cryptographic hashes (commitments) that guarantee the underlying data tables remain untampered. These commitments are essential for later phases, where our SXT Chain prover nodes will employ our Proof of SQL protocol to validate the integrity of the indexed data during ZK-proven SQL execution for client queries.

#### Testnet Target Timelines, subject to change
* **October 30: Testnet Phase 0 Begins**
  * Spin up the chain and test basic block building.
  * Verify client transaction signatures and data insertion by Ethereum indexer nodes.
  * Update, store, and retrieve cryptographic commitments for indexed data.
* **November 20: Testnet Phase 1 Begins**
  * Focus on advanced cryptography applied to DeFi data.
  * Implement self-service utilities for clients to request new data tables.
  * Enable developers to execute ZK-proven SQL queries via our Gateway API.
  * Test threshold signatures among validators and proof verification using Chainlink.
* **January 1: Testnet Phase 2 Begins**
  * Test complex chain features, including substrate verification and table creation for new commitments.
  * Establish proof verification on SXT Chain and query relaying via Substrate.
  * Introduce ZKpay payments for gas, with on-chain verification.

## I. Prerequisites
### 1.0 Terms of Testnet Participation
Please review the below instructions and confirm your interest in participating by replying to message you received from Space and Time with the following information:

* Name of the **entity** participating in the SXT Chain testnet
* Name of the **person** that will review and accept the terms of service for participating in the SXT Chain testnet
* **Email** of the person that will review and accept the Terms
* **Discord** handles / usernames for you or members of your team to be issued the Testnet Node Operator role in the [Space and Time Discord server](https://discord.com/invite/spaceandtimeDB).

Note that you will not be permissioned to join the SXT Chain test network until you have signed the Terms of Service via DocuSign that will be sent to the email provided above.

#### Incentives and Expectations
* **Participation Requirements**: Validators are expected to actively participate, with a response SLA of 48 hours for notifications around actions needed from validator operators, chain upgrades, changes to testing schedule, etc.
* **Incentive Structure**: We may choose to provide rewards to eligible Testnet participants for completing certain activities, including running nodes, performing other critical services related to the SxT Chain, and meeting certain performance criteria or other requirements. Additional details will be provided to you from time to time via Testnet-related websites and developer documentation.

**Discord information**: We have set up a Testnet Nodes channel in the SXT discord (https://discord.com/invite/spaceandtimeDB) to manage all communications and Q&A with node operators during testnet.

![image1](./assets/image1.png)

![image2](./assets/image2.png)


You will only be able to access this channel list if you hold the “Testnet Node Operator” role in the SXT Discord. Please share any team members’ Discord usernames with us directly via email at testnet@spaceandtime.io and we will assign this role to their profile(s).

### 1.1. System Specifications
The minimum system requirements for running a SXT validator node are shown in the table below:

| Key | Value
| --- | ---
| CPU cores | 16
| CPU Architecture | amd64
| Memory (GiB) | 128
| Storage (TiB) | 1
| Storage Type | SSD
| OS | Linux


On Azure cloud, this is equivalent to SKU `Standard_L16as_v3` with storage SKU of `PremiumV2_SSD`.

### 1.2. Downloads
#### 1.2.1. Docker Image
Assuming Docker Desktop is installed and working on your computer. The SXT Node Docker image can be downloaded with `docker pull` command or used directly with Helm chart.

```bash
docker pull ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0
docker images --digests ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0
```

When each new docker image is released we will also be sharing the full `sha256` hash of the image. Please confirm that hash against the image pulled down by docker with an extra docker `images` argument `--digests` to make sure that you are pulling the right one.

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.53.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

#### 1.2.2. Testnet Chainspecs
SXT testnet chainspecs are part of the docker images mentioned in [section 1.2.1](#121-docker-image). To inspect the chainspecs that comes with the docker image, please run the following:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  --entrypoint=bash ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  -c "cat /opt/chainspecs/testnet-spec.json"
```

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.53.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

#### 1.2.3. Helm Chart

> [!NOTE]
> If you choose to use Docker instead of Kubernetes, you may skip this step.

For installation on Kubernetes we have created a helm chart sxt-node-chart. Adding Helm repository with following command:

```bash
helm repo add sxt-charts https://spaceandtimelabs.github.io/sxt-node-helm-charts
```

#### 1.2.4. Validator Snapshots

To speed up the time it takes for a validator to sync up with the network, we have snapshots ready for consumption. You can download them with the following command:

```bash
wget https://blocks.testnet.sxt.network/snapshots/2024-11-20/sxt-testnet.tar.gz
wget https://blocks.testnet.sxt.network/snapshots/2024-11-20/SHA256SUMS
```
Alternatively, you can download the snapshot from the Stakecraft team's [dedicated page](https://services.stakecraft.com/docs/snapshots/substrate-sxt-snapshot)


> [!TIP]
> Due to the size of the snapshot, it can take a while for `wget` to download its contents. Other tools can be used to speed that up, if your internet connection permits, e.g. `aria2c -s16 -x16 <url>`

After that, verify the download checksum:
```bash
sha256sum -c SHA256SUMS
```

### 1.3. Testnet Bootnodes
Bootnodes on SXT networks are trusted peers on the network that a new node will first connect to and find more peers to download blocks from. The three bootnodes listed below are hosted by Space and Time:

```
/dns/bootnode0.testnet.sxt.network/tcp/30333/p2p/12D3KooWDV5kmYUR5nxruFBfdGX2ZMR43iSe3SfmopZ3sLBFvZzc
/dns/bootnode1.testnet.sxt.network/tcp/30333/p2p/12D3KooWGAQAg7iZgyn8wnnT8nkDo9NVAPbfubpMgL1mYTRNgjdD
/dns/bootnode2.testnet.sxt.network/tcp/30333/p2p/12D3KooWLLf8tW3PPbj9MCda9rfypNN5xyZRi1bKoLj8s9UkeJDZ
```

### 1.4. Wallet and Keys
#### 1.4.1. Wallet Seed Phrase
Here we demonstrate generating common 12-words seed phrases for validator wallets using the SXT Node Docker Image:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  --entrypoint=/usr/local/bin/sxt-node ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  key generate -w 12
```

All key information should now be available from the return of the command.  Store the information in a safe place.

> [!IMPORTANT]
> To continue with the next steps, take the value of `Secret seed:` and save it as an environment variable:
```bash
export SECRET_SEED=0x... # place the value of your Secret seed here
```

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.53.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

#### 1.4.2. Validator Keys
Note that there are TWO keys required. First is a block assignment participation key in aura format, the other is finality voting key in gran format. Note that as mentioned in [section 1.2.2](#122-testnet-chainspecs), SXT testnet chainspecs is part of the docker image, located in `/opt/chainspecs/testnet-spec.json` inside the container.

We will first create a folder to place those key output files before running the command in docker with the folder mounted:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  -v sxt-validator-key:/data \
  --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  key insert --scheme sr25519 --keystore-path /data \
  --chain /opt/chainspecs/testnet-spec.json --key-type aura \
  --suri "${SECRET_SEED?}"
```

Similarly, the voting key generation is:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  -v sxt-validator-key:/data \
  --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  key insert --scheme ed25519 --keystore-path /data \
  --chain /opt/chainspecs/testnet-spec.json --key-type gran \
  --suri "${SECRET_SEED?}"
```


Note that in addition to the `--key-type`, the `--scheme` is actually different in these commands for different keys. The `${SECRET_SEED}` here is the seed phrase generated in 1.4.1.

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.53.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

#### 1.4.3. Validator Node Key
A validator node key is used to create a node's peer id in order to uniquely identify the node over the p2p network. We first create a folder where we want to store the node-key, then mount the folder into docker image and run the key generating command:

```bash
docker run -it --rm \
  --platform linux/amd64 \
  -v sxt-node-key:/data \
  --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  key generate-node-key --chain /opt/chainspecs/testnet-spec.json --file /data/subkey.key
```

The generated key should now be in a file called `subkey.key` in the folder. Note that from the command line output it should also show you the peer id of the node.

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.53.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

### II. SXT Testnet Validator Onboarding
By setting up the SXT Testnet node using provided binaries, chainspecs, and bootnodes, the node will be able to connect to the SXT Testnet and start syncing blocks. However, since the SXT Chain testnet will leverage a permissioned Proof of Authority, in order to become an actual validator on the SXT Testnet, additional steps as listed below are required:

### 2.1. Generate Public Key Addresses
Similar to the two types of validator keys generated in [section 1.4.2](#142-validator-keys), the wallet seed phrase or secret seed that was created in [section 1.4.1](#141-wallet-seed-phrase) can also be used to generate two different public keys that are associated with the two types of validator keys:

The first CLI command uses schema `sr25519`, which creates public keys associated with the `aura` key:

```bash
docker run -it --rm --platform linux/amd64 --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  key inspect --scheme sr25519 "${SECRET_SEED?}"
```

The second CLI command uses schema `ed25519`, which creates public keys associated with the `gran` key:

```bash
docker run -it --rm --platform linux/amd64 --entrypoint=/usr/local/bin/sxt-node \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  key inspect --scheme ed25519 "${SECRET_SEED?}"
```

Both commands will generate output that looks something like:

```
Secret Key URI `******` is account:
  Network ID:        substrate
  Secret seed:       *******
  Public key (hex):  0x8eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48
  Account ID:        0x8eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48
  Public key (SS58): 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
  SS58 Address:      5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
```

Please record the `HEX` and `SS58` format of public key from both outputs (total 4 addresses). These are the public key addresses of your validator wallet.

> [!IMPORTANT]
> Do not share the contents of your secret seed with anyone.

> [!NOTE]
> Note: While the above references the `sxt-node:testnet-v0.53.0` docker image, this will change; please reference the "Resources" channel in the Testnet Nodes section of the [SXT Discord](https://discord.gg/spaceandtimeDB) or this [GitHub repository](https://github.com/orgs/spaceandtimelabs/packages/container/package/sxt-node) for the latest docker image.

### 2.2. Submit Addresses and Confirm Testnet Validator Participation
**Please send the public key addresses generated in [section 2.1](#21-generate-public-key-addresses) to us** (testnet@spaceandtime.io). In this note, please **confirm that the Discord handles affiliated with your team have already joined the [SXT Discord](https://discord.gg/spaceandtimeDB)**. Once keys are submitted and handles have been assigned the testnet NOP role, coordinate with the Space and Time team in Discord to get permissioned!

## III. Validator Setup Using Azure AKS

If you choose to use Docker instead of AKS, go to [IV. Validator Setup Using Docker](#iv-validator-setup-using-docker)

### 3.1. Azure AKS Setup Requirements
There are a few considerations when using Azure AKS to host SXT Testnet validator:

1. Due to P2P traffic considerations (more on this in [section 3.3](#33-important-notes-on-p2p-traffics-on-azure-aks)), Each AKS cluster can only host a single instance of SXT validator. Other node types may be able to share the same cluster.

1. Need to also allocate one Azure public IP (PIP) resource to be used by the validator node.

1. Validator deployment should be on its own AKS node pool, and configure the helm chart with proper `nodeSelector` label. This dedicated node pool should have proper VM SKU as described in [section 1.1](#11-system-specifications).

1. The validator seed phrase or secret seed generated in [section 1.4.1](#141-wallet-seed-phrase), as well as the node key generated in [section 1.4.3](#143-validator-node-key) should both be installed into a single Kubernetes secret on the cluster within the same namespace where the SXT node helm chart would be installed.

### 3.2. Configure Helm Values and Installation
Assuming the helm repo of SXT node chart has been set up already as described in [section 1.2.3](#123-helm-chart). We can start preparing helm chart configuration values.yaml as follow:

```yaml
---
networkID: testnet

sxt:
  isValidator: true
  image:
    repository: ghcr.io/spaceandtimelabs/sxt-node
    pullPolicy: IfNotPresent
    tag: "testnet-v0.53.0"
  genesisPath: "/opt/chainspecs/testnet-spec.json"
  secret:
    name: "validator-data"
    key: "Secret-Seed"
    nodeKey: "Node-Key"
  nodeSelectors:
    agentpool: validator-nodepool
  resources:
    storage: 4Ti
    cpu: 13000m
    memory: 24G

ingress-nginx:
  enabled: true
  Controller:
    Kind: DaemonSet
    ingressClassByName: true
    ingressClass: "nginxP2pSXT"
    service:
      annotations:
        service.beta.kubernetes.io/azure-pip-name: "azure-pip-testnet-sandbox-wus2-1"
        service.beta.kubernetes.io/azure-load-balancer-resource-group: "azure-rg-testnet-sandbox-wus2"
  tcp:
    "30333": "sxt-testnet/sxtnode-p2p-tcp-service:30333"
  udp:
    "30333": "sxt-testnet/sxtnode-p2p-udp-service:30333"
```

In this example configuration above, it is assumed that the helm will be deployed in namespace `sxt-testnet`, and the validator secret-seed (data key `Secret-Seed`) and node-key (data key `Node-Key`) will be in the kubernetes secret named `validator-data` within that namespace. As mentioned before, the SXT testnet chain chainspecs is distributed as part of the container, so the YAML entry `sxt.genesisPath` merely points to the chainspec that we will be using for the testnet.

The two annotations for the dependency chart, `ingress-nginx`, are to specify the public ip (pip) allocated for the AKS cluster as discussed in [section 2.1](#21-generate-public-key-addresses). The load-balancer resource group should point to the Azure resource group where the AKS belongs to. We also assigned a dedicated ingressClass for all traffic associated with this load-balancer. Finally, this example assumes that you have created a dedicated AKS node pool named `validator-nodepool` and the deployment will have a `nodeSelector` that selects nodes in the pool to deploy the validator.

Once the Helm configuration is done, we can now run the following command with proper `KUBECONFIG` to install the chart:

```
helm upgrade --install sxt-testnet-validator sxt-charts/sxt-node-chart --version=0.3.9 \
  -n sxt-testnet --create-namespace -f ./values.yaml --dependency-update
```

### 3.3. Important Notes on P2P traffics on Azure AKS
In order for validator P2P traffic to have same inbound and outbound IP for the validator, one will need to follow the following [this document](https://learn.microsoft.com/en-us/azure/aks/load-balancer-standard#:~:text=Update%20the%20cluster%20with%20your,IDs%20of%20your%20public%20IPs.&text=The%20above%20command%20shows%20the,IDs%20from%20the%20previous%20command) from Microsoft to configure the AKS outbound LB to have matching IP address of the inbound P2P LB (acting like a TCP traffic proxy).

## IV. Validator Setup Using Docker

If you choose to use AKS instead of Docker, go to [III. Validator Setup using AKS](#iii-validator-setup-using-azure-aks).

Using Docker image we can launch the validator without deploying Kubernetes cluster like Azure AKS:

Here we assume the setup uses the following volumes: `sxt-testnet-data` is the block storage volume, and `sxt-validator-key` is the volume where the two formats of validator keys are located.
Finally, the volume where the generated node key is stored is `sxt-node-key`.

### 4.1 Copy snapshot

To speed up initialization, it is recommended to download a snapshot of the data. Follow the instructions in [1.2.4 Validator Snapshots](#124-validator-snapshots)
and then run the commands below:

```bash
# Stop validator if running
docker ps -q --filter 'volume=sxt-testnet-data' | xargs --no-run-if-empty docker stop

# Start a new temporary container and mount volume
docker run -d -it --rm --name copy-data \
  --platform linux/amd64 \
  -v sxt-testnet-data:/data \
  --entrypoint=/bin/bash ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0

# Copy snapshot into container and extract data
docker exec -ti copy-data sh -c 'rm -rf /data/chains/sxt-testnet && mkdir -p /data/chains'
docker cp sxt-testnet.tar.gz copy-data:/data/chains/
docker exec -ti copy-data sh -c 'tar xf /data/chains/sxt-testnet.tar.gz -C /data/chains && rm -f /data/chains/sxt-testnet.tar.gz'
docker stop copy-data
```

### 4.2. Docker Run
```bash
docker run -d --restart always \
  --platform linux/amd64 \
  -v sxt-testnet-data:/data \
  -v sxt-validator-key:/key \
  -v sxt-node-key:/node-key \
  -p 30333:30333/tcp \
  -p 9615:9615/tcp \
  -p 9944:9944/tcp \
  ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0 \
  --base-path /data \
  --prometheus-port 9615 \
  --pool-limit 10240 \
  --pool-kbytes 1024000 \
  --chain /opt/chainspecs/testnet-spec.json \
  --keystore-path /key \
  --node-key-file /node-key/subkey.key \
  --bootnodes "/dns/bootnode0.testnet.sxt.network/tcp/30333/p2p/12D3KooWDV5kmYUR5nxruFBfdGX2ZMR43iSe3SfmopZ3sLBFvZzc" \
  --bootnodes "/dns/bootnode1.testnet.sxt.network/tcp/30333/p2p/12D3KooWGAQAg7iZgyn8wnnT8nkDo9NVAPbfubpMgL1mYTRNgjdD" \
  --bootnodes "/dns/bootnode2.testnet.sxt.network/tcp/30333/p2p/12D3KooWLLf8tW3PPbj9MCda9rfypNN5xyZRi1bKoLj8s9UkeJDZ" \
  --validator \
  --port 30333 \
  --log info \
  # only do the following if local RPC is desired
  --rpc-external \
  --unsafe-rpc-external \
  --rpc-methods unsafe \
  --rpc-cors all \
  --rpc-port 9944
```

### 4.3. Docker Compose
Prepare a `docker-compose.yaml` file as follows:

```yaml
---
name: 'sxt-testnet-node'

services:
  sxt-testnet:
    platform: linux/amd64
    restart: unless-stopped
    image: ghcr.io/spaceandtimelabs/sxt-node:testnet-v0.53.0
    ports:
      - '9615:9615' # metrics
      - '9944:9944' # rpc
      - '30333:30333' # p2p
    volumes:
      - sxt-testnet-data:/data
      - sxt-validator-key:/key
      - sxt-node-key:/node-key
    pid: host
    command: >
      --base-path /data
      --prometheus-port 9615
      --pool-limit 10240
      --pool-kbytes 1024000
      --chain /opt/chainspecs/testnet-spec.json
      --keystore-path /key
      --node-key-file /node-key/subkey.key
      --bootnodes "/dns/bootnode0.testnet.sxt.network/tcp/30333/p2p/12D3KooWDV5kmYUR5nxruFBfdGX2ZMR43iSe3SfmopZ3sLBFvZzc"
      --bootnodes "/dns/bootnode1.testnet.sxt.network/tcp/30333/p2p/12D3KooWGAQAg7iZgyn8wnnT8nkDo9NVAPbfubpMgL1mYTRNgjdD"
      --bootnodes "/dns/bootnode2.testnet.sxt.network/tcp/30333/p2p/12D3KooWLLf8tW3PPbj9MCda9rfypNN5xyZRi1bKoLj8s9UkeJDZ"
      --validator
      --port 30333
      --log info
      # only do the following if local RPC is desired
      --rpc-external
      --unsafe-rpc-external
      --rpc-methods unsafe
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
