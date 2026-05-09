# Deploying 3x-ui on Coolify

This setup uses one container. Docker Compose is used because Coolify can read the build context, restart policy, host networking, environment variables, and persistent volumes from one file.

## Choose one deployment mode

There are two valid Coolify modes for this app:

- `docker-compose.coolify.yml`: best for VPN usage. Uses host networking. Open the panel with `http://YOUR_SERVER_IP:2053`, not through Coolify's app URL.
- `docker-compose.coolify-proxy.yml`: best if you need `https://your-domain` to open the panel through Coolify. You must publish every VPN inbound port manually.

Do not use a Coolify domain/proxy with `network_mode: host`. Coolify's proxy expects a Docker-network upstream, and host-network containers are not reachable that way. If you combine them, Coolify can show errors such as `no server running`.

## Recommended VPN setup

1. Create a new Coolify resource from this repository.
2. Choose **Docker Compose** as the build pack.
3. Set the compose file to:

   ```text
   docker-compose.coolify.yml
   ```

4. Deploy the resource.
5. Open the panel at:

   ```text
   http://YOUR_SERVER_IP:2053
   ```

Do not set a public Coolify domain for this host-network mode. If you already set one, remove it from the Coolify resource and use the server IP plus port.

The default 3x-ui credentials are:

```text
username: admin
password: admin
```

Change the username, password, panel port, and base path immediately after the first login.

## Why host networking is used

3x-ui is a VPN/Xray control panel, not only a web application. The panel listens on port `2053` by default, but the VPN inbounds you create later can use many different TCP or UDP ports.

With normal Docker port mappings, every inbound port must be published before it can receive traffic. Host networking lets Xray bind directly to the VPS network, so new inbounds work without editing the Coolify service each time.

## Persistent data

The Coolify compose file defines named volumes for:

- `xui-db`: panel database and settings at `/etc/x-ui`
- `xui-cert`: panel certificates at `/root/cert`
- `xui-logs`: logs at `/var/log/x-ui`

Back up `xui-db` before deleting the Coolify resource or Docker volumes.

## Fail2ban

`XUI_ENABLE_FAIL2BAN` is disabled by default in `docker-compose.coolify.yml`.

Fail2ban inside a container needs extra network/firewall permissions to manage host firewall rules. If you want to use it, prefer configuring Fail2ban directly on the VPS host. Only enable container Fail2ban if you understand the firewall permissions it needs.

## Domain/proxy setup

If you want `https://v2ray.vpn.wardksa.com` to open the 3x-ui panel through Coolify, use:

```text
docker-compose.coolify-proxy.yml
```

In Coolify, set the service domain to route to container port `2053`. In the domain field, enter the domain with the container port suffix:

```text
https://v2ray.vpn.wardksa.com:2053
```

This mode uses normal Docker networking, so every VPN inbound port must be added to the `ports:` list before traffic can reach it.

Example:

```yaml
services:
  3xui:
    build: .
    restart: unless-stopped
    expose:
      - "2053"
    ports:
      - "2053:2053/tcp"
      - "2096:2096/tcp"
      - "443:443/tcp"
      - "443:443/udp"
    volumes:
      - xui-db:/etc/x-ui
      - xui-cert:/root/cert
      - xui-logs:/var/log/x-ui
    environment:
      TZ: Asia/Riyadh
      XRAY_VMESS_AEAD_FORCED: "false"
      XUI_ENABLE_FAIL2BAN: "false"
      SERVICE_FQDN_3XUI_2053: "https://v2ray.vpn.wardksa.com:2053"

volumes:
  xui-db:
  xui-cert:
  xui-logs:
```

Use the fixed-port version only when all inbound ports are known and stable.
