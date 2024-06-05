# NEAR RPC Speedup project

## Introduction

The `near-rpc-speedup` project is designed to enhance the efficiency of NEAR protocol RPC nodes by introducing a cache storage mechanism. This added communication layer enables faster distribution of chunks among nodes, optimizing data exchange and improving overall network performance.

This repository contains two critical services:
- [**Caching Proxies Cache**](https://github.com/aurora-is-near/caching-proxies-cache): A web service facilitating chunk retrieval.
- [**Caching Proxies Terminal**](https://github.com/aurora-is-near/caching-proxies-terminal): Manages the submission of chunks to the cache.

## Deployment Guide

This guide walks you through deploying these services on your machine (where the validators are located) and outlines the necessary setup changes.

### Services Overview

Both services are written in Go, offering two deployment options:
1. **Build from Source**: Clone the repositories and build the services using Go. This approach is suitable for those preferring to compile the latest versions themselves.
2. **Docker Images**: For convenience, pre-built Docker images are available, and `docker-compose` files are provided for easy deployment.

### Docker Deployment (Recommended)

Using Docker and `docker-compose` is the preferred method for deploying these services due to its simplicity and ease of use. Below is a brief explanation of the provided `docker-compose` configuration.

#### Docker Compose Configuration for testnet

```yaml
services:
  cache:
    image: nearaurora/caching-proxies-cache:latest
    container_name: caching-proxies-cache-public-testnet
    pull_policy: always
    command:
      - "./app"
      - "-shard-prefix=shards"
      - "-shards-to-listen=1,2,3,4"
      - "-creds=config/testnet.creds"
      - "-server=nats://rpc-speedup-nats.testnet.aurora.dev"
    volumes:
      - ./config:/app/config
    restart: unless-stopped
    ports:
      - "1324:1324"

  terminal:
    image: nearaurora/caching-proxies-terminal:latest
    container_name: caching-proxies-terminal-public-testnet
    pull_policy: always
    command:
      - "./app"
      - "-server=nats://rpc-speedup-nats.testnet.aurora.dev"
      - "-submissions-verifier-host=https://rpc-speedup-verifier.testnet.aurora.dev/authenticate"
      - "-shard-prefix=shards"
    volumes:
      - ./config:/app/config
    restart: unless-stopped
    ports:
      - "1323:1323"

  watchtower:
    image: containrrr/watchtower
    container_name: caching-proxies-watchtower
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    logging:
      driver: "json-file"
      options:
        max-size: 5m
        max-file: "5"
    command: --interval 30 caching-proxies-cache-public-testnet caching-proxies-terminal-public-testnet
```

**Note**: The `config/testnet.creds` file is located is located [here](https://github.com/aurora-is-near/near-rpc-speedup/config/testnet.creds`)

- **Cache Service**: Manages the retrieval of chunks. It listens on specified shards and interacts with a NATS server for communication.
- **Terminal Service**: Handles chunk submissions to the cache and verifies submissions through an authentication service.
- **Watchtower**: Automatically updates running Docker containers to the latest version, ensuring your services stay up-to-date without manual intervention.

Step-by step guide on setup:
1. Identify home directory for a rpc-speedup project(Ex. `mkdir -p /var/rpc-speedup` )
2. Place docker-compose.yml file in the rpc-speedup directory. You can download the file from [here]()

#### NEAR Validator Configuration

To integrate these services with your NEAR validator node, you need to modify the `config.json` of the NEAR node to include the following configuration:

```json
"chunk_distribution_network": {
  "enabled": true,
  "uris": {
    "get": "http://host:1324/get",
    "set": "http://host:1323/process?token=[token]"
  }
}
```

Replace `host` with the actual hostname or IP address where the services are running. The `[token]` placeholder should be replaced with a token provided by us for each validator instance.

### Getting Started

1. **Configure**: Create the `docker-compose.yml`. Make sure that `testnet.creds` file is available.
2. **Launch Services**: Run `docker-compose up -d` to start the services.
3. **Update NEAR `config.json`**: Modify your NEAR node's configuration file as described above to enable communication with the caching services. Do not forget to restart nodes after these changes.

## Support

For support, feature requests, or to provide feedback, please open an issue in the respective GitHub repository. If you need direct assistance or have specific requirements (e.g., obtaining a token for your validator instance), feel free to contact us directly.