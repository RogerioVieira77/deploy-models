# Checklist - sistema-de-condominios

## 1. Identificacao da aplicacao

- Nome da aplicacao: sistema-de-condominios
- Repositorio GitHub: `git@github.com:RogerioVieira77/sistema-de-condominios.git`
- Owner tecnico: a definir
- Owner funcional: a definir
- Linguagem: Python + TypeScript
- Framework: FastAPI + Next.js + PostgreSQL + Redis

---

## 2. Ambientes e dominios

- DEV: a definir
- HOM: a definir
- VALIDACAO: a definir
- PROD: a definir

---

## 3. Runtime e empacotamento

- [x] Possui `Dockerfile`
- [x] Possui arquivo de compose do ambiente
- [x] Possui `.env.example`
- [x] Possui healthcheck
- [ ] Possui smoke test
- [x] Possui endpoint ou log de versao implantada

Observacoes:

- `docker-compose.staging.yml` foi ajustado para bind local em frontend, backend, postgres e redis.
- O compose foi endurecido para homologacao: backend sem `--reload` e frontend usando stage `runner`.
- Healthchecks identificados em `/health/live`, `/health/ready` e endpoint de meta em `/api/v1/meta`.
- Existem testes backend e frontend, mas nao foi identificado smoke test operacional por ambiente.
- Runtime ativo atualizado em 2026-03-20 com validacao local bem-sucedida em `127.0.0.1:8000` e `127.0.0.1:3000`.

---

## 4. Rede e exposicao

- [x] Banco nao esta publicado externamente
- [x] Redis ou cache nao esta publicado externamente
- [x] Backend nao esta publicado externamente sem justificativa
- [x] Servico exposto no host usa `127.0.0.1`
- [ ] NGINX configurado como entrada publica unica
- [x] UFW revisado

Observacoes:

- O bind agora esta local, mas ainda falta vhost NGINX e politica de publicacao por dominio.
- O frontend em `127.0.0.1:3000` ainda requer camada de proxy publico para homologacao real.
- Nao existe dominio de homologacao explicitamente documentado no repositorio nem arquivo NGINX preexistente no host para esta aplicacao.

---

## 5. Dados e dependencias

- [x] Banco identificado
- [ ] Politica de backup definida
- [x] Variaveis de ambiente mapeadas
- [x] Integracoes externas mapeadas
- [ ] Credenciais fora do repositorio

Observacoes:

- Banco PostgreSQL e Redis locais identificados.
- Variaveis de auth, storage e banking estao mapeadas em `.env.example`.

---

## 6. Pipeline GitHub Actions

- [ ] Workflow de CI implementado
- [ ] Workflow de build de imagem implementado
- [ ] Workflow de deploy DEV implementado
- [ ] Workflow de deploy HOM implementado
- [ ] Workflow de deploy VALIDACAO definido
- [ ] Workflow de deploy PROD definido
- [ ] GitHub Environments configurados
- [ ] Secrets por ambiente configurados

Observacoes:

- Nenhum workflow foi identificado localmente neste repositorio durante a analise.

---

## 7. Promocao e rollback

- [ ] Estrategia de branches definida
- [ ] Estrategia de tags definida
- [ ] Versao implantada fica registrada
- [ ] Rollback por tag anterior documentado
- [ ] Procedimento de incidente documentado

Observacoes:

- A aplicacao esta tecnicamente mais preparada para pipeline do que `polidrama`, mas ainda depende de formalizar deploy por ambiente.

---

## 8. Criterios de pronto

- [x] Aplicacao pronta para DEV
- [ ] Aplicacao pronta para HOM
- [ ] Aplicacao pronta para VALIDACAO
- [ ] Aplicacao pronta para PROD

Responsavel pela avaliacao: GitHub Copilot

Data: 2026-03-20