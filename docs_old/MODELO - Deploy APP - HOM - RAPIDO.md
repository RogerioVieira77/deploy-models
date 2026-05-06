# ⚡ MODELO RÁPIDO — Deploy APP — HOM (1 página)

## 0) Variáveis (preencha e execute)

```bash
APP_NAME="<nome-app>"
APP_DOMAIN="<subdominio>.unicomunitaria.com.br"
APP_REPO_PATH="/opt/unicomunitaria/docker/<pasta-projeto>"
APP_PORT="<8081-8099>"
```

---

## 1) Pré-check

```bash
cd "$APP_REPO_PATH"
sudo ss -tulnp
docker ps
ufw status verbose
```

---

## 2) Ajustes mínimos de código/config

### 2.1 `docker-compose.yml` (padrão seguro)
- banco sem porta pública
- backend sem porta pública
- frontend publicado só localmente:

```yaml
ports:
  - "127.0.0.1:${APP_PORT}:80"
```

### 2.2 Frontend (API same-origin)

```js
const API_URL = '/api'
```

---

## 3) Build + subir containers

```bash
cd "$APP_REPO_PATH"
docker compose up -d --build
docker compose ps

curl -sS -o /dev/null -w '%{http_code}\n' "http://127.0.0.1:${APP_PORT}/"
curl -sS "http://127.0.0.1:${APP_PORT}/health"
```

---

## 4) NGINX (vhost)

```bash
cat > "/etc/nginx/sites-available/${APP_NAME}.hml.conf" <<EOF
server {
    listen 80;
    server_name ${APP_DOMAIN};
    location / { return 301 https://\$host\$request_uri; }
}

server {
    listen 443 ssl http2;
    server_name ${APP_DOMAIN};

    ssl_certificate /etc/letsencrypt/live/unicomunitaria.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/unicomunitaria.com.br/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:${APP_PORT};
        proxy_http_version 1.1;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }
}
EOF

ln -sfn "/etc/nginx/sites-available/${APP_NAME}.hml.conf" "/etc/nginx/sites-enabled/${APP_NAME}.hml.conf"
nginx -t && systemctl reload nginx
```

---

## 5) Padronizar HTTPS para evitar warning no `nginx -t`

```bash
# auditar listens 443
grep -RIn --include='*.conf' 'listen .*443' /etc/nginx

# padrão recomendado em todos os vhosts HTTPS:
# listen 443 ssl http2;

nginx -t && systemctl reload nginx
```

---

## 6) Hardening (firewall)

```bash
# bloquear porta da app externamente
ufw deny ${APP_PORT}/tcp

# opcional: limpar duplicidades de 80/443 (mantendo Nginx Full)
ufw delete allow 80 || true
ufw delete allow 443 || true
ufw delete allow 80/tcp || true
ufw delete allow 443/tcp || true

ufw status numbered
```

---

## 7) Validação final pública

```bash
curl -I "http://${APP_DOMAIN}" | head -n 5
curl -I "https://${APP_DOMAIN}" | head -n 10
curl -sS "https://${APP_DOMAIN}/health"
curl -sS -o /dev/null -w '%{http_code}\n' "https://${APP_DOMAIN}/api/usuarios"
```

### Resultado esperado
- HTTP → `301` para HTTPS
- HTTPS → `200`
- `/health` → status `OK`
- endpoint API principal → `200` (ou código esperado da app)
