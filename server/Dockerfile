FROM node:16

WORKDIR /usr/src/app

COPY . .

RUN npm ci --no-audit || npm ci --no-audit --maxsockets 1

 EXPOSE 8080

CMD [ "npm", "start" ]
