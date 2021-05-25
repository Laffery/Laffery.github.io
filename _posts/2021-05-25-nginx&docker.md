---
title: 使用Nginx和Docker部署Next.ts项目
date: 2021-05-25 14:04
categories: [Note, FE]
tags: [nginx, docker]
comments: true
---

## 背景

1. 前段时间在字节的课程上接触了Next.js框架，觉得很好用，另一方面也想锻炼一下TypeScript能力，为暑期在字节实习做准备

        我称之为 Next.ts

2. 一直想做一个属于我的网站，这些天受到一些东西的熏染，决定开动了

## Getting started

首先初始化我们的项目，需要加入ts支持

> create-next-app [name] --example with-typescript

然后部署到我的服务器上

但是这还没完，这个项目跑在3000端口，而**我们平常访问的网站都是访问默认端口80**，于是我想到了使用`Nginx反向代理`-大概的原理就是Nginx监听80端口客户端发来的请求，然后作为代理发送给3000端口

## Nginx配置

本部分参考[文章](https://steveholgado.com/nginx-for-nextjs/)

具体的做法就是使用Docker构建专属于本项目的配置文件`default.conf`

    ```conf
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=STATIC:10m inactive=7d use_temp_path=off;

    upstream nextjs_upstream {
    server nextjs:3000;
    }

    server {
    listen 80 default_server;
    
    server_name _;

    server_tokens off;

    gzip on;
    gzip_proxied any;
    gzip_comp_level 4;
    gzip_types text/css application/javascript image/svg+xml;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;

    location / {
        proxy_pass http://nextjs_upstream;
    }
    }
    ```

Dockerfile:

    ```Dockerfile
    # Base on offical NGINX Alpine image
    FROM nginx:alpine

    # Remove any existing config files
    RUN rm /etc/nginx/conf.d/*

    # Copy config files
    # .conf files in conf.d/ dir get included in main config
    COPY ./default.conf /etc/nginx/conf.d/

    # Expose the listening port
    EXPOSE 80

    # Launch NGINX
    CMD [ "nginx", "-g", "daemon off;" ]
    ```

## Next Docker 配置

Dockerfile:

    ```Dockerfile
    FROM node:14

    # Set working directory
    WORKDIR /usr/app

    # Copy package.json and package-lock.json before other files
    # Utilise Docker cache to save re-installing dependencies if unchanged
    COPY ./package*.json ./

    # Install dependencies
    RUN npm install --production

    # Copy all files
    COPY ./ ./

    RUN yarn add --dev typescript @types/react @types/node

    # Build app
    RUN npm run build

    # Expose the listening port
    EXPOSE 3000

    # Run container as non-root (unprivileged) user
    # The node user is provided in the Node.js Alpine base image
    USER node

    # Run npm start script when container starts
    CMD [ "npm", "start" ]
    ```

## Deploy

    ```sh
    # cd work_dir

    # build
    docker build -t [next-image] .
    docker build -t [nginx-image] ./nginx/

    docker network create [network]
    docker run -d --network [network] --name [next-name] [next-image]
    docker run -d --network [network] --link [next-name]:nextjs -p 80:80 --name [nginx-name] [nginx-image]
    ```
