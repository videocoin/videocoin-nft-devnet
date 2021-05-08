FROM node:13

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install app dependencies

RUN npm install --global node-gyp
RUN npm install truffle -g
