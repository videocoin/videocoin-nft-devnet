FROM vidnft-base as wyvern-ethereum

# set the working direction
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# add app
COPY ./pkgs/wyvern-ethereum-experiments ./pkgs/wyvern-ethereum-experiments

WORKDIR /app/pkgs/wyvern-ethereum-experiments
RUN yarn

# deploy wyvern-ethereum contracts 

ARG ARG_PRIV_KEY=""
ENV VID_PRIV_KEY=$ARG_PRIV_KEY
ARG ARG_RPC=""
ENV VID_RPC=$ARG_RPC


RUN truffle --config truffle.js --network viddev migrate
RUN ls -al


FROM vidnft-base as wyver-js
WORKDIR /app
COPY ./pkgs/wyvern-js ./pkgs/wyvern-js


COPY --from=wyvern-ethereum /app/pkgs/wyvern-ethereum-experiments/config.json /app/pkgs/wyvern-js/src/wyvern-ethereum/config.json
COPY --from=wyvern-ethereum /app/pkgs/wyvern-ethereum-experiments/build /app/pkgs/wyvern-js/src/wyvern-ethereum/build

WORKDIR /app/pkgs/wyvern-js

RUN yarn --verbose
RUN scripts/build.sh

FROM vidnft-base as front

WORKDIR /app

RUN npm install react-scripts@3.4.1 -g

# add app
COPY ./package.json ./package.json
COPY ./public ./public
COPY ./src ./src
COPY ./pkgs/Opensea-js  ./pkgs/Opensea-js
COPY ./pkgs/wyvern-schemas-experiments ./pkgs/wyvern-schemas-experiments
COPY --from=wyvern-js /app/pkgs/wyvern-js /app/pkgs/wyvern-js

RUN yarn
#RUN ls -la /
# start app
CMD ["npm", "start"]