# PORTAS_EM_USO

Levantamento realizado em 23/02/2026 com os comandos:

- `sudo ss -tulnp`
- `sudo ss -tulnp | grep 80`
- `docker ps`

## Padrão atual do servidor

| Faixa | Uso |
|---|---|
| 80 | NGINX HTTP |
| 443 | NGINX HTTPS |
| 8081–8099 | Aplicações Docker (range reservado) |

## Portas em uso (estado atual)

| Aplicação/Serviço | Porta | Origem | Processo/Container |
|---|---:|---|---|
| nginx (proxy HTTP) | 80 | Host | nginx |
| nginx (proxy HTTPS) | 443 | Host | nginx |
| sistema-de-enderecos-api | 8000 | Docker publicado | `sistema-enderecos-api` |
| polidrama_backend | 8091 | Docker publicado | `polidrama_backend` |
| polidrama_frontend | 8092 | Docker publicado | `polidrama_frontend` |
| polidrama_db | 15432 | Docker publicado | `polidrama_db` |
| SSH | 22 | Host | sshd |
| DNS local (resolved) | 53 | Host (loopback) | systemd-resolved |

## Containers e portas (docker ps)

| Container | Portas |
|---|---|
| sistema-enderecos-api | 0.0.0.0:8000->8000/tcp |
| sistema-enderecos-db | 5432/tcp (sem publicação externa) |
| polidrama_backend | 0.0.0.0:8091->8000/tcp |
| polidrama_frontend | 0.0.0.0:8092->80/tcp |
| polidrama_db | 0.0.0.0:15432->5432/tcp |

## Observações operacionais

- O proxy oficial público continua sendo o NGINX nas portas 80/443.
- As aplicações Docker estão publicadas em portas altas para roteamento interno/operação.
- Regra de segurança desejada: portas de app (808x/809x) acessíveis somente localmente/rede interna; exposição externa deve ser bloqueada por firewall.
