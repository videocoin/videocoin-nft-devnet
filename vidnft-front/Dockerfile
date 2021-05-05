FROM node:14  

# set the working direction
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install app dependencies
COPY package.json ./
# add app
COPY ./pkgs ./pkgs

#COPY package-lock.json ./

#RUN apk add --no-cache git
#RUN npm install
RUN yarn
RUN npm install react-scripts@3.4.1 -g

# add app
COPY . ./
RUN ls -la /

# start app
CMD ["npm", "start"]