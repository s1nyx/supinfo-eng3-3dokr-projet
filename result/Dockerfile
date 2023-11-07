FROM node:18-alpine3.17

ENV NODE_ENV production

WORKDIR /usr/src/app

COPY package*.json ./

RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci --only=production

USER node

COPY --chown=node:node . ./

EXPOSE 8888

CMD ["npm", "start"]