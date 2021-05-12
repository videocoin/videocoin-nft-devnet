# Video NFT Devnet

Docker-compose based devnet that includes the following services:
* Frontend hosting
* API Backend
* Content Management
* Blockchain devnet

![Video NFT Devenet](./docs/devnet.drawio.svg)

## Frontend

Frontend is created using a layered image as shown below:  
![Frontend hosting service, docker image build-time](./docs/frontend.drawio.svg)

The repo includes the follwoing Opesea SDK pacakges as submodules. These modules are modified during build time of the docker image based on the user configuration.

An example to build the front-end image supplying private key and web3 rpc for deployment of exchange contracts:

Base image containing truffle:
```
cd vidnft-base
docker build -t vidnft-base .
```
```
cd vidnft-front
docker build --build-arg ARG_PRIV_KEY=0xXXXXXXXX --build-arg ARG_RPC=http://localhost:8545 --network="host" -t vidnft-front.
```
This produces an image that contains wywvern-js containing config.json and build folders of wyvern-ethereum with the newly deployed contract addresses, ABI.

### wywvern-ethereum 
The wyvern exchange contracts are installed to the selected blockchain during build-time of the image.
### wyvern-js
The package is updated with deployed wyvern-ethereum contract addresss(config.json) and build folders from the previous layer to this package during build time.

### Opensea-js
The package includes the wyvern-js as a local folder

## API Backend (vidnft-oderbook)
vidnft-oderbook wraps the backend mockup. See the details of the backend in the following repo:    
https://github.com/videocoin/opensea-js-test

## Marketplace
This service provides the backend API for the VideoCoin NFT.

Testing the service docker image:

### Obtain TextileHub credentials
https://docs.textile.io/hub/apis/

### Obtain google credentials for transcoder service

https://cloud.google.com/transcoder/docs/quickstart

### Start postgress server
```
docker run -e POSTGRES_HOST_AUTH_METHOD=trust -e POSTGRES_USER=root  -e POSTGRES_DB=marketplace --name postgres-docker  -p 5432:5432  postgres
```
Example environment variables file (env.list)
```
MARKETPLACE_TEXTILE_AUTH_KEY=bkhxdigv2moryg2ic4gom3erhra
MARKETPLACE_TEXTILE_AUTH_SECRET=bcvtedbv6eg76cxftzltq7jgseo7te3hamyesgay
MARKETPLACE_TEXTILE_THREAD_ID=bafkyofsunajdhdjyah5n4ui2ryxzlhetnwx25fe2d3iq5vbgk4mqacq
MARKETPLACE_TEXTILE_BUCKET_ROOT_KEY=bafzbeieuvodl4ypexptlmkfmreq23x66l6djn5depvde7r5jstgstau2nm
MARKETPLACE_BLOCKCHAIN_URL=http://localhost:8545
MARKETPLACE_ERC1155_CONTRACT_ADDRES=0xCfEB869F69431e42cdB54A4F4f105C19C080A601
MARKETPLACE_ERC1155_CONTRACT_KEY={"address":"90f8bf6a479f320ead074411a4b0e7944ea8c9c1","crypto":{"cipher":"aes-128-ctr","ciphertext":"309d59afeb0fe19859d17d3e6e6c182782ec85523fbd60a05b702452c5757e1b","cipherparams":{"iv":"14888e5290d7fbc4eecebbf6b966ef0a"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"e3bb675ae8a06a6c7048d4a264894cf456fc2275ad76aa7e61f9b09623dab379"},"mac":"3e5555aa211c3267dcb94724fd4164923e37b1301eac33c2eb0af1e8c8535f63"},"id":"7c02ca35-80a6-4178-be3d-56e1224fc3e1","version":3}
MARKETPLACE_ERC1155_CONTRACT_KEY_PASS=vidnfttest
GOOGLE_APPLICATION_CREDENTIALS=/home/ram/liveplanet/videocoin-nft-demo/gcp-credentials.json
MARKETPLACE_GCP_BUCKET=videocoin-nft-demo-1
```
### Start marketplace service
```
docker run -it --rm --env-file ./env.list --network host marketplace
```
### API Commands (Examples)
### Registration
Sample:
```
curl -X 'POST' 'http://127.0.0.1:8088/api/v1/accounts' -H 'accept: application/json'  -H 'Content-Type: application/json' -d '{"address": "0xFFcf8FDEE72ac11b5c542428B35EEF5769C409f0"}'
```
## Blockchain devnet (vcndev)
vcndev sevice includes a 3-node Ethereum PoA network.

## Demo steps of the Video NFT Installer
(These are intended steps. Not completed yet)
* Configure the docker-compose.yaml (Owner wallet, Textile Hub Credentials)
* Run the installer. docker-compose up -d
* Create test accounts (User_A, User_B) and fund
* mint a Video NFT using User_A account
  * Inputs: video_file, token_id
  * Outputs: token_URI and encrypted video assets
  * Operation:
    * Upload video_file(Frontend) => Encrypt(CMS) => Upload to Textile Hub
    * Upload thumbnail to Textile Hub(CMS)
    * Upload Token_URI File to Textile Hub(CMS)
    * Mint NFT token with token_id and token_URI(FrontEnd=>Blockchain))
  * Play the video with User_A DRM data(FrontEnd)
* Make SellOrder with User_A account
  * Inputs: Token Contract address, token_Id, ask amount
  * Operation:
    * Add SellOrder (OrderBook)
* Fulfill the order with User_B account
  * Inputs: Token Contract address, token_Id, offer amount
  * Outputs: Updated Token_Uri 
  * Outputs: token_URI and encrypted video assets
  * Operation:
    * Update order(OrderBook)
    * ReEncrypt(CMS) => Upload to Textile Hub
    * Upload Token_URI File to  Textile Hub
    * Transfer NFT token to User_B (Blockchain)
  * Play the video with User_B DRM Data

### User Registration with marketplace
![User Registration](./docs/user-registration.svg)
### NFT Minting Sequence Diagram
![NFT Minting Sequence Diagram](./docs/mint-sequence.svg)