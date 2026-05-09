# Deploying 3x-ui on Coolify

This setup uses one container. Docker Compose is used because Coolify can read the build context, restart policy, host networking, environment variables, and persistent volumes from one file.

## Recommended Coolify setup

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

## Fixed-port alternative

If your Coolify server does not allow `network_mode: host`, you can use normal Docker port mappings, but you must list every panel, subscription, and VPN inbound port in advance.

Example:

```yaml
services:
  3xui:
    build: .
    restart: unless-stopped
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

volumes:
  xui-db:
  xui-cert:
  xui-logs:
```

Use the fixed-port version only when all inbound ports are known and stable.
