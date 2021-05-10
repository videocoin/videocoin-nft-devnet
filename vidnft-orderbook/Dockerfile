FROM node:14  

# set the working direction
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

COPY package.json ./
RUN npm install
# add app
COPY Opensea-js/examples/backend-mock/  ./
RUN ls -la /app

# start app
CMD ["node", "server.js"]