---
title: ''
emoji: '🌟'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

```docker
FROM node:16-alpine

RUN npm i -g @nestjs/cli

WORKDIR /web/src

COPY package.json /web/src
COPY yarn.lock /web/src

RUN yarn install

COPY . /web/src

CMD [ "yarn", "start:dev" ]
```
