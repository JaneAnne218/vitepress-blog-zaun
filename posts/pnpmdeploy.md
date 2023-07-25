---
title: pnpmdeploy
description: pnpmdeploy
date: 2023-6-10
tags:
  - CI/CD
---
## 写一个pnpm的github action
```deploy.yml
name: Deployhahah

on:
  push:
    branches:
      branch-1

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: 16
          registry-url: https://registry.npm.taobao.org
      - uses: pnpm/action-setup@v2
        with:
          version: latest
      - run: pnpm install
      - run: pnpm run build
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: docs/.vitepress/dist
```