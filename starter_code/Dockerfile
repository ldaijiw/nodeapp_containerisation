FROM node as APP

LABEL MAINTAINER=lwaltmann@spartaglobal.com

# Create app directory
WORKDIR /usr/src/app

COPY app/ .

RUN npm install

FROM node:alpine

COPY --from=app /usr/src/app /usr/src/app

WORKDIR /usr/src/app

EXPOSE 3000

CMD ["npm", "run", "seed;"]

CMD ["npm", "start", "app.js;"]
