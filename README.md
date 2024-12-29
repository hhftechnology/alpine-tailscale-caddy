# Tailscale-Caddy Proxy

[![Docker Image CI/CD](https://github.com/hhftechnology/alpine-tailscale-caddy/actions/workflows/docker-publish.yml/badge.svg?branch=main)](https://github.com/hhftechnology/alpine-tailscale-caddy/actions/workflows/docker-publish.yml)

A Docker image that seamlessly enables sharing of HTTP services over the Tailscale network with automatic HTTPS support. This solution combines the power of Tailscale's secure networking with Caddy's automated certificate management to provide a robust, maintenance-free way to expose web services to authorized users.

## Overview

In today's containerized environments, sharing web services securely often involves complex configuration of firewalls, authentication systems, and SSL certificates. This project simplifies that process by leveraging Tailscale's secure networking capabilities and Caddy's automatic HTTPS features.

### Key Features

The Tailscale-Caddy proxy provides several advantages over alternative solutions:

- Automatic SSL certificate management with zero configuration
- Seamless service container restarts without affecting the proxy
- Simple environment variable-based configuration
- Secure access limited to authorized Tailscale users
- Support for multiple services and domains
- Zero-trust security model through Tailscale's network

## How It Works

The proxy container runs two main components:

1. A Tailscale daemon that connects your service to the Tailscale network
2. A Caddy server that handles HTTPS termination and proxying

When started, the container:

1. Establishes a connection to your Tailscale network
2. Generates a Caddy configuration based on your environment variables
3. Automatically obtains and manages SSL certificates through Tailscale
4. Proxies incoming requests to your service container

## Prerequisites

Before using this proxy, ensure you have:

1. A Tailscale account with an active tailnet
2. HTTPS and MagicDNS enabled in your Tailscale admin console
3. Docker and Docker Compose installed on your host system
4. Basic familiarity with Docker networking concepts

## Configuration Options

### Essential Environment Variables

- `TS_HOSTNAME`: The device name in your Tailscale network
  Example: `myapp-proxy`

- `TS_TAILNET`: Your tailnet name (without the .ts.net suffix)
  Example: `mycompany`

- `CADDY_TARGET`: The service endpoint to proxy to
  Example: `webapp:8080`

### Optional Parameters

- `TS_EXTRA_ARGS`: Additional arguments for the Tailscale daemon
  Example: `--advertise-exit-node --hostname=custom-name`

### Volume Mounts

- `/var/lib/tailscale`: Stores Tailscale credentials (required for persistence)

## Usage Scenarios

### Scenario 1: Basic Web Application

This example shows how to expose a simple web application:

```yaml
version: '3'

networks:
  app_network:
    external: false

volumes:
  tailscale_state:

services:
  webapp:
    image: nginx
    volumes:
      - ./website:/usr/share/nginx/html
    networks:
      - app_network

  tailscale_proxy:
    image: hhftechnology/alpine-tailscale-caddy:latest
    volumes:
      - tailscale_state:/var/lib/tailscale
    environment:
      - TS_HOSTNAME=webapp-proxy
      - TS_TAILNET=mycompany
      - CADDY_TARGET=webapp:80
    networks:
      - app_network
    restart: always
    init: true
```

### Scenario 2: Multiple Services

You can run multiple instances of the proxy to expose different services:

```yaml
version: '3'

networks:
  internal_network:
    external: false

volumes:
  ts_state_app1:
  ts_state_app2:

services:
  app1:
    image: ghost:latest
    networks:
      - internal_network

  app2:
    image: wordpress:latest
    networks:
      - internal_network

  ts_proxy_app1:
    image: hhftechnology/alpine-tailscale-caddy:latest
    volumes:
      - ts_state_app1:/var/lib/tailscale
    environment:
      - TS_HOSTNAME=blog-proxy
      - TS_TAILNET=mycompany
      - CADDY_TARGET=app1:2368
    networks:
      - internal_network

  ts_proxy_app2:
    image: hhftechnology/alpine-tailscale-caddy:latest
    volumes:
      - ts_state_app2:/var/lib/tailscale
    environment:
      - TS_HOSTNAME=wordpress-proxy
      - TS_TAILNET=mycompany
      - CADDY_TARGET=app2:80
    networks:
      - internal_network
```

### Scenario 3: Development Environment

Perfect for local development with multiple services:

```yaml
version: '3'

networks:
  dev_network:
    external: false

volumes:
  ts_dev_state:

services:
  frontend:
    image: node:latest
    command: npm run dev
    volumes:
      - ./frontend:/app
    networks:
      - dev_network

  backend:
    image: python:3.9
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./backend:/app
    networks:
      - dev_network

  ts_proxy:
    image: hhftechnology/alpine-tailscale-caddy:latest
    volumes:
      - ts_dev_state:/var/lib/tailscale
    environment:
      - TS_HOSTNAME=dev-environment
      - TS_TAILNET=mycompany
      - CADDY_TARGET=frontend:3000
    networks:
      - dev_network
```

## Setup Process

1. Create your docker-compose.yml file using one of the examples above
2. Start the containers:
   ```bash
   docker-compose up
   ```
3. Look for the authentication URL in the proxy container logs
4. Visit the URL to authenticate the device
5. In the Tailscale admin console:
   - Disable key expiry for the device
   - Configure access controls as needed
6. Restart the containers in detached mode:
   ```bash
   docker-compose up -d
   ```

## Accessing Your Services

After setup, your services will be available at:
- `https://[TS_HOSTNAME].[TS_TAILNET].ts.net`
- `http://[TS_HOSTNAME].[TS_TAILNET].ts.net` (automatically redirects to HTTPS)

For example:
- `https://webapp-proxy.mycompany.ts.net`

## Troubleshooting

Common issues and solutions:

1. Certificate errors:
   - Ensure HTTPS is enabled in your Tailscale admin console
   - Verify MagicDNS is enabled
   - Check the Caddy logs: `docker-compose logs ts_proxy`

2. Connection issues:
   - Confirm the service container is running
   - Verify network configuration
   - Check Tailscale device status in admin console

3. Authentication problems:
   - Re-authenticate using the URL in logs
   - Check key expiry settings
   - Verify ACL permissions

## Security Considerations

The Tailscale-Caddy proxy provides several security benefits:

1. Zero-trust network access through Tailscale
2. Automatic HTTPS encryption
3. Access control through Tailscale ACLs
4. Isolated Docker networks
5. No exposed ports on the host machine

## Advanced Usage

### Custom Tailscale Configuration

You can pass additional arguments to Tailscale using `TS_EXTRA_ARGS`:

```yaml
environment:
  - TS_EXTRA_ARGS=--hostname=custom-name --advertise-exit-node --advertise-tags=tag:web
```

### Network Isolation

Create separate networks for different service groups:

```yaml
networks:
  frontend_net:
    internal: true
  backend_net:
    internal: true
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Original Tailscale Docker image team
- Caddy web server project
- lpasselin for the initial inspiration
- The Tailscale community
