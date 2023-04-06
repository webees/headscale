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
cat << EOF > /app/headscale/compose.yaml
version: '3.8'

services:
  headscale:
    container_name: headscale
    image: headscale/headscale:latest
    restart: unless-stopped
    networks:
      - traefik
    volumes:
      - ./:/etc/headscale
    command: headscale serve
    labels:
      - traefik.enable=true
      - traefik.http.routers.headscale.entrypoints=https
      - traefik.http.routers.headscale.tls=true
      - traefik.http.routers.headscale.rule=Host(`p2p.dev.run`)
      - traefik.http.services.headscale.loadbalancer.server.port=8080
  headscale_ui:
    container_name: headscale_ui
    image: ghcr.io/gurucomputing/headscale-ui:latest
    restart: unless-stopped
    networks:
      - traefik
    labels:
      - traefik.enable=true
      - traefik.http.routers.headscale_ui.entrypoints=https
      - traefik.http.routers.headscale_ui.tls=true
      - traefik.http.routers.headscale_ui.middlewares=default
      - traefik.http.routers.headscale_ui.rule=Host(`p2p.dev.run`) && PathPrefix(`/web`)
      - traefik.http.services.headscale_ui.loadbalancer.server.port=80

networks:
  traefik:
    external: true
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
