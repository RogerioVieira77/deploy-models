# Deploy em Homologação - Sistema de Endereços

## Estado aplicado em 23/02/2026 (modo assistido)

- Caminho em uso no servidor: `/opt/unicomunitaria/docker/sistema-de-enderecos`
- Stack aplicada com sucesso via `make bootstrap`
- API validada localmente em `127.0.0.1:8000`
- NGINX configurado com vhost `sistema-de-enderecos.hml.conf`
- HTTPS habilitado usando certificado wildcard já existente:
	- `/etc/letsencrypt/live/unicomunitaria.com.br/fullchain.pem`
	- `/etc/letsencrypt/live/unicomunitaria.com.br/privkey.pem`
- Endpoint público validado:
	- `https://sistema-de-enderecos.unicomunitaria.com.br/health`
	- `https://sistema-de-enderecos.unicomunitaria.com.br/geo/integracao/cadastro/cep/57071170`

## Objetivo

Publicar a aplicação no subdomínio:

- https://sistema-de-enderecos.unicomunitaria.com.br

Fluxo principal em homologação:

- App Cadastro chama `GET /geo/integracao/cadastro/cep/{cep}`
- Sistema de Endereços retorna:
	- `logradouro`
	- `bairro`
	- `cidade`
	- `estado`
	- `latitude`
	- `longitude`

---

## 1) Pré-requisitos do servidor

Executar como usuário com permissão de `sudo`.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release git make nginx certbot python3-certbot-nginx
```

Instalar Docker Engine + plugin Compose (caso ainda não esteja instalado):

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
	"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
	$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
	sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

Após adicionar usuário ao grupo docker, reabra a sessão SSH.

Validar ferramentas:

```bash
docker --version
docker compose version
nginx -v
```

---

## 2) DNS e acesso de rede

Antes de publicar:

- Criar/confirmar registro DNS `A` para `sistema-de-enderecos.unicomunitaria.com.br` apontando para IP do servidor de homologação.
- Liberar portas no firewall:
	- `22/tcp` (SSH)
	- `80/tcp` (HTTP - emissão/renovação SSL)
	- `443/tcp` (HTTPS)

Se usar UFW:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

---

## 3) Estrutura da aplicação no servidor

Caminho já utilizado em homologação:

```bash
cd /opt/unicomunitaria/docker/sistema-de-enderecos
```

Caso precise provisionar outro servidor do zero:

```bash
sudo mkdir -p /opt/unicomunitaria/docker
sudo chown -R $USER:$USER /opt/unicomunitaria/docker
cd /opt/unicomunitaria/docker
git clone git@github.com:RogerioVieira77/sistema-de-enderecos.git
cd sistema-de-enderecos
```

Observação importante:

- Este projeto já possui `Dockerfile`, `docker-compose.yml`, `Makefile`, schema SQL e bootstrap ETL.
- Não é necessário recriar estes arquivos no deploy.

---

## 4) Subir stack da homologação (DB + API + carga inicial)

Executar bootstrap completo:

```bash
cd /opt/unicomunitaria/docker/sistema-de-enderecos
make bootstrap
```

O comando acima realiza:

1. Sobe container `db` (`postgis/postgis:16-3.4`)
2. Build da imagem da API FastAPI (`python:3.12-slim`)
3. Executa `python -m scripts.bootstrap` para:
	 - garantir database `sistema_enderecos`
	 - aplicar schema `sql/001_init.sql`
	 - carregar CSV `arquivos_carga/CEPs_BRASIL.csv`
4. Sobe o container `api` (uvicorn em `0.0.0.0:8000`)

Verificar containers:

```bash
docker compose ps
```

Verificar logs:

```bash
make logs
```

---

## 5) Validar API localmente no servidor

Healthcheck:

```bash
curl -i http://127.0.0.1:8000/health
```

Endpoint de integração (Cadastro):

```bash
curl -i http://127.0.0.1:8000/geo/integracao/cadastro/cep/57071170
```

Validação automática do contrato:

```bash
./scripts/test_cadastro_endpoint.sh 57071170
```

Se o CEP testado não existir na base e retornar 404, buscar um válido no banco:

```bash
docker compose exec -T db psql -U postgres -d sistema_enderecos -t -A -c \
"SELECT cep FROM endereco_cep LIMIT 1;"
```

---

## 6) Publicar via NGINX (reverse proxy)

Criar arquivo de site:

```bash
sudo nano /etc/nginx/sites-available/sistema-de-enderecos.hml.conf
```

Conteúdo:

```nginx
server {
	listen 80;
	server_name sistema-de-enderecos.unicomunitaria.com.br;

	location / {
		return 301 https://$host$request_uri;
	}
}

server {
	listen 443 ssl http2;
	server_name sistema-de-enderecos.unicomunitaria.com.br;

	ssl_certificate /etc/letsencrypt/live/unicomunitaria.com.br/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/unicomunitaria.com.br/privkey.pem;

	location / {
		proxy_pass http://127.0.0.1:8000;
		proxy_http_version 1.1;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
	}
}
```

Ativar site e validar sintaxe:

```bash
sudo ln -sf /etc/nginx/sites-available/sistema-de-enderecos.hml.conf /etc/nginx/sites-enabled/sistema-de-enderecos.hml.conf
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7) Habilitar HTTPS (Let's Encrypt)

Status atual aplicado neste servidor:

- O domínio já utiliza certificado wildcard válido (`*.unicomunitaria.com.br`).
- Não foi necessário emitir novo certificado específico para este subdomínio.

Validar certificados instalados:

```bash
sudo certbot certificates
```

Se em outro servidor não houver certificado válido para o domínio, emitir:

```bash
sudo certbot --nginx -d sistema-de-enderecos.unicomunitaria.com.br
```

Verificar renovação automática:

```bash
sudo systemctl status certbot.timer
sudo certbot renew --dry-run
```

---

## 8) Testes finais (externos)

No próprio servidor ou de uma máquina externa:

```bash
curl -i https://sistema-de-enderecos.unicomunitaria.com.br/health
curl -i https://sistema-de-enderecos.unicomunitaria.com.br/geo/integracao/cadastro/cep/57071170
```

Esperado:

- `/health` retorna `200` e `{"status":"ok"}`
- `/geo/integracao/cadastro/cep/{cep}` retorna `200` e os campos:
	- `logradouro`, `bairro`, `cidade`, `estado`, `latitude`, `longitude`

---

## 9) Operação diária

Comandos úteis:

```bash
cd /opt/unicomunitaria/docker/sistema-de-enderecos
make up
make down
make logs
```

Ver status de containers:

```bash
docker compose ps
```

---

## 10) Atualização de versão (redeploy)

```bash
cd /opt/unicomunitaria/docker/sistema-de-enderecos
git pull origin main
make bootstrap
```

Notas:

- `make bootstrap` reaplica schema (idempotente) e recarrega CSV inicial.
- Para atualização sem recarga de base, usar:

```bash
make up
```

---

## 11) Rollback rápido

Se a versão nova falhar:

```bash
cd /opt/unicomunitaria/docker/sistema-de-enderecos
git log --oneline -n 5
git checkout <commit_anterior_estavel>
make bootstrap
```

---

## 12) Checklist de pronto para homologação

- [x] DNS do subdomínio apontando para servidor
- [x] Docker/Compose/NGINX instalados
- [x] Projeto disponível em `/opt/unicomunitaria/docker/sistema-de-enderecos`
- [x] `make bootstrap` executado sem erro
- [x] `curl /health` retornando `200`
- [x] Endpoint de integração testado com CEP real
- [x] Script `./scripts/test_cadastro_endpoint.sh` validando contrato
- [x] NGINX ativo com proxy para `127.0.0.1:8000`
- [x] HTTPS ativo (certificado wildcard válido ou cert emitido no host)
- [x] Teste externo em `https://sistema-de-enderecos.unicomunitaria.com.br`