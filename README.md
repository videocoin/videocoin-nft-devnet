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
### Configure DB
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
docker run -it --rm --env-file ./env.list -name marketplace --network host marketplace
```
```
docker exec -it marketplace-c '/goose -dir /migrations -table marketplace postgres "host=127.0.0.1 port=5432 dbname=marketplace sslmode=disable" up'
```

### API Commands (Examples)
### Registration
Sample:
```
curl -X 'POST' 'http://127.0.0.1:8088/api/v1/accounts' -H 'accept: application/json'  -H 'Content-Type: application/json' -d '{"address": "0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1"}'
```

Get nonce:
```
curl -X 'GET' \
  'http://127.0.0.1:8088/api/v1/accounts/0xFFcf8FDEE72ac11b5c542428B35EEF5769C409f0/nonce' \
  -H 'accept: application/json'
```

Generate Signature, using nonce as msg  
python script:
```
from web3.auto import w3
from eth_account.messages import encode_defunct
private_key="0x4f3edf983ac636a65a842ce7c78d9aa706d3b113bce9c46f30d7d21715b23b1d"
message = encode_defunct(text="4wtXGUGcE2AjqDFzs8bG")
signed_msg = w3.eth.account.sign_message(message, private_key)
signed_msg.signature
# Output 
HexBytes('0x78e18ec88f7f89a8be7bbe2c69d71696f3fe1ae781b460f3911dcc9a7b9359a1063e4903bfce5451407031852c9b8e3c205ed0d192b0377f521565acead43a241b')
```
Authenticate:
```
curl -X 'POST' 'http://127.0.0.1:8088/api/v1/auth' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{
  "address": "0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1",
  "signature": "0x78e18ec88f7f89a8be7bbe2c69d71696f3fe1ae781b460f3911dcc9a7b9359a1063e4903bfce5451407031852c9b8e3c205ed0d192b0377f521565acead43a241b"
}'
```
Upload Video / Mint Token
```
curl -X 'POST' \
  'http://127.0.0.1:8088/api/v1/assets/upload' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MjA5NzEyNjUsInN1YiI6IjIiLCJpc19hY3RpdmUiOmZhbHNlLCJhZGRyZXNzIjoiMHg5MGY4YmY2YTQ3OWYzMjBlYWQwNzQ0MTFhNGIwZTc5NDRlYThjOWMxIn0.aRzMlZk5rMUdP9ivsJ4Vr1rMnh59Fxr7L7cac8rYaI8' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@bb_test_1080p_120s.mp4;type=video/mp4'
```
Response:
```
{
  "id": 1,
  "token_id": null,
  "name": null,
  "description": null,
  "content_type": "video/mp4",
  "status": "PROCESSING",
  "thumbnail_url": null,
  "preview_url": null,
  "encrypted_url": null,
  "yt_video_id": null,
  "owner": null,
  "asset_contract": null
}
```

#### Get Asset
/api/v1/assets/{id}

```
curl -X 'GET' \
  'http://127.0.0.1:8088/api/v1/assets/1' \
  -H 'accept: application/json'
```
Response:
```
{
  "id": 1,
  "token_id": "1",
  "name": null,
  "description": null,
  "content_type": "video/mp4",
  "status": "READY",
  "thumbnail_url": "https://hub.textile.io/ipns/bafzbeieuvodl4ypexptlmkfmreq23x66l6djn5depvde7r5jstgstau2nm/a/2/BiLzlw-1620885298821766747/thumb.jpg",
  "preview_url": "https://hub.textile.io/ipns/bafzbeieuvodl4ypexptlmkfmreq23x66l6djn5depvde7r5jstgstau2nm/a/2/BiLzlw-1620885298821766747/original.mp4",
  "encrypted_url": "https://hub.textile.io/ipns/bafzbeieuvodl4ypexptlmkfmreq23x66l6djn5depvde7r5jstgstau2nm/a/2/BiLzlw-1620885298821766747/encrypted.mp4",
  "yt_video_id": null,
  "owner": {
    "id": 2,
    "address": "0x90f8bf6a479f320ead074411a4b0e7944ea8c9c1",
    "profile_img_url": null,
    "user": {
      "username": null,
      "name": null
    }
  },
  "asset_contract": {
    "address": "",
    "name": "",
    "description": "",
    "asset_contract_type": "",
    "schema_name": "ERC1155",
    "symbol": "",
    "buyer_fee_basis_points": 0,
    "seller_fee_basis_points": 250,
    "opensea_buyer_fee_basis_points": 0,
    "opensea_seller_fee_basis_points": 250,
    "dev_buyer_fee_basis_points": 0,
    "dev_seller_fee_basis_points": 0
  }
}
```

#### Get Asset (contract + toke_Id)
/api/v1/asset/{contract_address}/{token_id}
```
curl -X 'GET' \
  'http://127.0.0.1:8088/api/v1/asset/0xCfEB869F69431e42cdB54A4F4f105C19C080A601/1' \
  -H 'accept: application/json'
```

Response:
```
{
  "message": "Not Found"
}
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