FROM vidnft-base as wyvern-ethereum

# set the working direction
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# add app
COPY ./pkgs/wyvern-ethereum-experiments ./pkgs/wyvern-ethereum-experiments

WORKDIR /app/pkgs/wyvern-ethereum-experiments
RUN yarn

#
# deploy wyvern-ethereum contracts 
#
ARG ARG_PRIV_KEY=""
ENV VID_PRIV_KEY=$ARG_PRIV_KEY
ARG ARG_RPC=""
ENV VID_RPC=$ARG_RPC


RUN truffle --config truffle.js --network viddev migrate
RUN ls -al

#
# Build wyvern-js with updated contracts
#
FROM vidnft-base as wyvern-js
WORKDIR /app

RUN npm install -g typescript

COPY ./pkgs/wyvern-js/package.json ./pkgs/wyvern-js/package.json
COPY ./pkgs/wyvern-js/tsconfig.json ./pkgs/wyvern-js/tsconfig.json
COPY ./pkgs/wyvern-js/webpack.config.js ./pkgs/wyvern-js/webpack.config.js
COPY ./pkgs/wyvern-js/scripts ./pkgs/wyvern-js/scripts
COPY ./pkgs/wyvern-js/src ./pkgs/wyvern-js/src
COPY --from=wyvern-ethereum /app/pkgs/wyvern-ethereum-experiments/config.json /app/pkgs/wyvern-js/src/wyvern-ethereum/config.json
COPY --from=wyvern-ethereum /app/pkgs/wyvern-ethereum-experiments/build /app/pkgs/wyvern-js/src/wyvern-ethereum/build

WORKDIR /app/pkgs/wyvern-js
RUN yarn --verbose
#RUN scripts/build.sh
# lint is giving problem, skif for now.

RUN cat package.json
RUN cat yarn.lock
RUN yarn const_gen
RUN yarn abi_gen
RUN npm install -g typescript

RUN yarn --verbose run tsc


#
# Fronend with updated wyvern-js
#
FROM vidnft-base as front

WORKDIR /app


RUN npm install react-scripts@3.4.1 -g

COPY ./package.json ./package.json
COPY ./pkgs/wyvern-schemas-experiments ./pkgs/wyvern-schemas-experiments

#copy updated wyvern-js
COPY --from=wyvern-js /app/pkgs/wyvern-js /app/pkgs/wyvern-js

COPY ./pkgs/Opensea-js  ./pkgs/Opensea-js
# Check if  we need to run "npm run build" in Opensea-js
RUN yarn

ENV NETWORK="vdddev"
ENV API_URL="http://localhost:3000"

ENV PATH /app/node_modules/.bin:$PATH
COPY ./public ./public
COPY ./src ./src
COPY ./.env ./.env

# start app
CMD ["yarn", "start"]