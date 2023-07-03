FROM node:14-alpine as builder

WORKDIR /app

COPY package.json .

RUN yarn install

COPY . .

RUN yarn run build

###################################

FROM nginx as runner

WORKDIR /usr/share/nginx/html

COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf

COPY --from=builder /assets/bundles .

EXPOSE 3000
