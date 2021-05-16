# Video NFT Devnet

## Quick start
checkout the repo, get the submodules and run the docker-compose.
```
git submodule update --init --recursive
```
Update the docker-compose to lattest vetsion(v1.29.1)  
https://docs.docker.com/compose/install/


Run the command

```
docker-compose up
```

Open the browser and launch the aft-app.
```
http://173.26.0.130/
```

Open Lightweight Block Explorer
```
http://173.26.0.140/
```

## VideoNFT Marketplace Components
Docker-compose based devnet that includes the following services:
* NFT-APP(Video NFT Frontend)
* Marketplace(API Backend)
* Postgres(Database for userinfo)
* Ganache(Blockchain Devnet)
* Explorer(Lightweight Block Explorer)
* Token Contracts(Contract deployer)

![Video NFT Devenet](./docs/devnet.drawio.svg)

## nft-app
VideoCoin NFT Frontend

Environment Variables
```
REACT_APP_BASE_URL=
REACT_APP_NETWORKS=
REACT_APP_TOKEN_ADDRESS=
REACT_APP_ESCROW_ADDRESS=
```
## Marketplace
This service provides the backend API for the VideoCoin NFT.

Testing the service docker image:

### Obtain TextileHub credentials
https://docs.textile.io/hub/apis/

### Configure Marketplace
Example environment variables file (env.list)
```
MARKETPLACE_TEXTILE_AUTH_KEY=
MARKETPLACE_TEXTILE_AUTH_SECRET=
MARKETPLACE_TEXTILE_THREAD_ID=
MARKETPLACE_TEXTILE_BUCKET_ROOT_KEY=
MARKETPLACE_BLOCKCHAIN_URL=
MARKETPLACE_ERC1155_CONTRACT_ADDRES=0
MARKETPLACE_ERC1155_CONTRACT_KEY=
MARKETPLACE_ERC1155_CONTRACT_KEY_PASS=
```

## Ganache
Test network.

Environment variables:
```
NETWORK_ID=
```
## Token Contracts
Environment variables:
```
VID_PRIV_KEY=""
VID_RPC=http://localhost:8545
ERC1155_TOKEN_URI_TEMPLATE="ipfs://{id}"
```

## Explorer
Lightweight Block Explorer
