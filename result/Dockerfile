# Stage 1
FROM node:18-alpine3.17 AS base

WORKDIR /usr/src/app

COPY package*.json ./

# Stage 2
FROM base AS dev

RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci --only=production

COPY . .

CMD ["npm", "start"]

# Stage 3
FROM base AS prod

ENV NODE_ENV production

RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci --only=production

USER node

COPY --chown=node:node . ./

EXPOSE 8888

CMD ["npm", "start"]