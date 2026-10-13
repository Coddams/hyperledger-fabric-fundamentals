## Hyperledger Fabric Fundamentals

Hands-on implementation of a permissioned blockchain network using Hyperledger Fabric, completed as foundational preparation for research into blockchain-based IoT device identity and security.

This repo documents a working, end-to-end Fabric deployment: a multi-organization network, a custom channel, deployed chaincode (smart contract), and a Node.js application interacting with it programmatically via the Fabric Gateway SDK. It follows Hyperledger's official `fabric-samples` tutorials, run and verified independently, with notes on what each stage does and why it matters for permissioned blockchain architectures in multi-party systems (e.g., IoT device registries, supply chains).

Research context: This work directly supports my ongoing project — a Decentralized IoT Device Identity System comparing permissioned (Fabric) and permissionless (Ethereum) architectures for lightweight device registration and revocation.

## Architecture

- **Network:** 2 organizations (Org1, Org2), each running one peer node, plus a shared Raft-based ordering service — Hyperledger Fabric's standard test-network topology
- **Channel:** `mychannel` — private communication layer joined by both organizations, each maintaining its own copy of the ledger
- **Chaincode:** `asset-transfer-basic` (Go) — deployed, approved by both organizations, and committed to the channel following Fabric's endorsement policy (both orgs must approve before any transaction is valid)
- **Client application:** Node.js app using the `@hyperledger/fabric-gateway` SDK — connects to the network programmatically (not via CLI) to submit and evaluate transactions

## Tech Stack

- Hyperledger Fabric v2.5.16
- Docker Desktop (network containerization)
- Go (chaincode language + Fabric binary tooling)
- Node.js v20+ with `@hyperledger/fabric-gateway` and `@grpc/grpc-js`
- Git Bash (Windows development environment)

## How to Run

Prerequisites: Docker Desktop, Git, Go, Node.js v20+, and `jq` installed.

1. Clone Hyperledger's official samples and pull Fabric's binaries/images:
   curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh
   chmod +x install-fabric.sh
   ./install-fabric.sh docker samples binary
   
2. Launch the network and create a channel:
   cd fabric-samples/test-network
   ./network.sh up
   ./network.sh createChannel -c mychannel
   
3. Deploy the sample chaincode:
   ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
   
4. Run the Node.js client application:
   cd ../asset-transfer-basic/application-gateway-javascript
   npm install
   npm start
   

Full tutorial reference: [Hyperledger Fabric docs](https://hyperledger-fabric.readthedocs.io/en/latest/)

## Results & Observations

Successfully executed the full transaction lifecycle:
- Initialized the ledger with 6 sample assets via chaincode invocation
- Created a new asset and transferred ownership between parties, with both changes correctly reflected on query
- Verified Fabric's endorsement policy in practice — transactions only commit once both Org1 and Org2 approve, enforced automatically by the network rather than by application logic
- Confirmed correct error handling: an update attempted against a non-existent asset was properly rejected and caught at the application layer, rather than silently corrupting ledger state

**Environment notes (Windows + Git Bash):**
- Git Bash's automatic path conversion interferes with Docker volume mounts during `network.sh up`; resolved by prefixing that specific command with `MSYS_NO_PATHCONV=1` rather than setting it globally, which would otherwise break `configtxgen`'s ability to locate its config file
- Container state (and the ledger inside it) persists across `docker stop`/`start`, but `network.sh down` performs a full reset — including ledger data and generated certificates — which is a meaningful distinction when resuming work across sessions

**Relevance to IoT device identity research:** This permissioned, multi-organization model — where transactions require endorsement from multiple parties before being committed — maps directly onto a plausible IoT device registry architecture: a device manufacturer and a network operator, for example, could jointly govern device onboarding and revocation without either party having unilateral control over the shared record.  