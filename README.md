# docker
```
apt update
apt install -y \
    curl \
    net-tools

curl -sSL https://get.docker.com/ | sh

apt install -y docker-compose-plugin
```

# headscale
```
mkdir -p /app/headscale

cd /app/headscale
```

```
cat << EOF > /app/headscale/Caddyfile
{
  skip_install_trust
  auto_https disable_redirects
  http_port {\$HTTP_PORT}
  https_port {\$HTTPS_PORT}
}

:{\$HTTP_PORT} {
  handle_path /web* {
    file_server {
      root /web
    }
  }
  reverse_proxy * http://headscale:8080
}
EOF
```

```
cat << EOF > /app/headscale/compose.yaml
version: '3.5'
services:
  headscale:
    container_name: headscale
    image: headscale/headscale:latest-alpine
    restart: unless-stopped
    volumes:
      - /app/headscale:/etc/headscale
    command: headscale serve
  headscale-ui:
    container_name: headscale-ui
    image: ghcr.io/gurucomputing/headscale-ui:latest
    restart: unless-stopped
    volumes:
      - /app/headscale/Caddyfile:/data/Caddyfile
    ports:
      - 8888:80
EOF
```

```
curl https://raw.githubusercontent.com/juanfont/headscale/main/config-example.yaml -o /app/headscale/config.yaml

sed -i "s/server_url: http:\/\/127.0.0.1:8080/server_url: http:\/\/${server_url}/g" /app/headscale/config.yaml
```

```
docker compose up -d
docker ps

docker exec headscale headscale apikeys create
```

# tailscale
```
chmod o+r /usr/share/keyrings/tailscale-archive-keyring.gpg
curl -fsSL https://tailscale.com/install.sh | sh

tailscale up --login-server http://${server_url} --authkey ${authkey} --force-reauth
```
