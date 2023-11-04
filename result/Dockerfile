FROM node:18-alpine3.17

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm ci --only=production

COPY . ./

EXPOSE 8888

CMD ["npm", "start"]