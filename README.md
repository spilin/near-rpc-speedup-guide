# NEAR RPC Speedup project

## Introduction

The `near-rpc-speedup` project is designed to enhance the efficiency of NEAR protocol RPC nodes by introducing a cache storage mechanism. This added communication layer enables faster distribution of chunks among nodes, optimizing data exchange and improving overall network performance.

This repository contains information on two critical services:
- [**Caching Proxies Cache**](https://github.com/aurora-is-near/caching-proxies-cache): A web service facilitating chunk retrieval.
- [**Caching Proxies Terminal**](https://github.com/aurora-is-near/caching-proxies-terminal): Manages the submission of chunks to the cache.

## Deployment Guide

This guide walks you through deploying these services on your machine (where the validators are located) and outlines the necessary setup changes.

### Services Overview

Both services are written in Go, recomended deployment is to build both services using the provided Dockerfiles and deploy them using `docker-compose`.

### Docker Deployment (Recommended)

Using Docker and `docker-compose` is the preferred method for deploying these services due to its simplicity and ease of use. Below is a brief explanation of the provided `docker-compose` configuration.

#### Docker Compose Configuration for testnet

```yaml
services:
  cache:
    image: caching-proxies-cache:latest
    container_name: caching-proxies-cache-testnet
    pull_policy: never
    command:
      - "./app/app"
      - "-shard-prefix=shards"
      - "-shards-to-listen=1,2,3,4"
      - "-creds=config/caching.creds"
      - "-server=nats://rpc-speedup-nats.testnet.aurora.dev"
    volumes:
      - ./config:/app/config
    restart: unless-stopped
    ports:
      - "1324:1324"

  terminal:
    image: caching-proxies-terminal:latest
    container_name: caching-proxies-terminal-testnet
    pull_policy: never
    command:
      - "./app/app"
      - "-server=nats://rpc-speedup-nats.testnet.aurora.dev"
      - "-submissions-verifier-host=https://rpc-speedup-verifier.testnet.aurora.dev/authenticate"
      - "-shard-prefix=shards"
    volumes:
      - ./config:/app/config
    restart: unless-stopped
    ports:
      - "1323:1323"
```

Step-by step guide on setup:
1. git clone [this]() repo 
1. Identify home directory for a rpc-speedup project and config directory. Ex.:
```bash
  git clone git@github.com:spilin/near-rpc-speedup-guide.git
```
2. Place docker-compose.yml file in the rpc-speedup directory. You can download the file from [here](https://github.com/spilin/near-rpc-speedup-guide/blob/master/docker-compose.yaml). Alternatively use curl command to download the file:
```bash
  curl -o docker-compose.yaml https://raw.githubusercontent.com/spilin/near-rpc-speedup-guide/master/docker-compose.yaml
```
3. You must request a `token` to authenticate with validator and `testnet.creds` to allow your cache instance to communicate with nats server. Please contact us directly to obtain a them.
4. Place `testnet.creds` file that you obtained from Aurora Labs in the config directory. 
5. Start the services using the following command:
```bash
  docker-compose up -d --force-recreate
```
6. To integrate these services with your NEAR validator node, you need to modify the `config.json` of the NEAR node to include the following configuration:

```json
"chunk_distribution_network": {
  "enabled": true,
  "uris": {
    "get": "http://caching-proxies-cache-testnet:1324/get",
    "set": "http://caching-proxies-terminal-testnet:1323/process?token=[token]"
  }
}
```

>Important:
Replace `caching-proxies-cache-testnet` and `caching-proxies-terminal-testnet` with the actual hostname or IP address where the services are running. With docker container setup could be as simple as `http://0.0.0.0`. The `[token]` placeholder should be replaced with a token provided by in step 3.

7. Restart the NEAR node to apply the changes.

### Getting Started

1. **Configure**: Create the `docker-compose.yml`. Make sure that `testnet.creds` file is available.
2. **Launch Services**: Run `docker-compose up -d` to start the services.
3. **Update NEAR `config.json`**: Modify your NEAR node's configuration file as described above to enable communication with the caching services. Do not forget to restart nodes after these changes.

## Support

For support, feature requests, or to provide feedback, please open an issue in the respective GitHub repository. If you need direct assistance or have specific requirements (e.g., obtaining a token for your validator instance), feel free to contact us directly.