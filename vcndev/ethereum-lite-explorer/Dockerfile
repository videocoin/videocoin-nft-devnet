FROM node:10.16 AS build

WORKDIR /build

RUN git clone https://github.com/Alethio/ethereum-lite-explorer.git .

RUN npm install

RUN npm i -g @alethio/cms-plugin-tool@^1.0.0-beta.5

#COPY . .

RUN npm run build --verbose

RUN acp install \
    @alethio/explorer-plugin-eth-common@4.0.0 \
    @alethio/explorer-plugin-eth-lite@4.2.0 \
    @alethio/explorer-plugin-eth-memento@2.0.2 \
    @alethio/explorer-plugin-eth-ibft2@2.0.1 \
    @alethio/explorer-plugin-3box@1.1.1

FROM nginx:stable-alpine

RUN apk update && \
    apk add nodejs

COPY --from=build /build/dist /usr/share/nginx/html
COPY --from=build /build/.docker/nginx.conf /etc/nginx/conf.d/default.conf

WORKDIR /app

#COPY config.default.json .
COPY --from=build /build/config.default.json /app/
#COPY set-env-vars.js .
COPY --from=build /build/set-env-vars.js /app/
#COPY .docker/entrypoint.sh .
COPY --from=build /build/.docker/entrypoint.sh /app/
RUN ls -al
ENTRYPOINT ["./entrypoint.sh"]

CMD ["nginx", "-g", "daemon off;"]