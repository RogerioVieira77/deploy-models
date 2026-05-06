# Checklist - polidrama

## 1. Identificacao da aplicacao

- Nome da aplicacao: polidrama
- Repositorio GitHub: `git@github.com:RogerioVieira77/polidrama.git`
- Owner tecnico: a definir
- Owner funcional: a definir
- Linguagem: Python + JavaScript
- Framework: FastAPI + React/Vite + PostgreSQL

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
- [ ] Possui endpoint ou log de versao implantada

Observacoes:

- `docker-compose.yml` foi endurecido para bind local em banco, backend e frontend.
- Healthcheck identificado em `/health`.
- Existem testes backend em `tests/`, mas nao foi encontrado smoke test operacional por dominio.
- O repositorio esta com worktree bastante sujo, incluindo arquivos de dados e banco versionados/modificados localmente.

---

## 4. Rede e exposicao

- [x] Banco nao esta publicado externamente
- [x] Redis ou cache nao esta publicado externamente
- [x] Backend nao esta publicado externamente sem justificativa
- [x] Servico exposto no host usa `127.0.0.1`
- [ ] NGINX configurado como entrada publica unica
- [x] UFW revisado

Observacoes:

- O compose foi ajustado para bind local, mas ainda nao ha evidencia de vhost NGINX ativo para a aplicacao.
- O estado sujo do repositorio e um bloqueador operacional para automatizar deploy com seguranca.

---

## 5. Dados e dependencias

- [x] Banco identificado
- [ ] Politica de backup definida
- [x] Variaveis de ambiente mapeadas
- [ ] Integracoes externas mapeadas
- [ ] Credenciais fora do repositorio

Observacoes:

- Banco PostgreSQL local via servico `db`.
- Ha forte acoplamento com diretorios de dados e artefatos locais dentro do repositorio.

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

- Antes de falar em rollback por release, e necessario limpar o worktree e separar dados operacionais do codigo fonte.

---

## 8. Criterios de pronto

- [ ] Aplicacao pronta para DEV
- [ ] Aplicacao pronta para HOM
- [ ] Aplicacao pronta para VALIDACAO
- [ ] Aplicacao pronta para PROD

Responsavel pela avaliacao: GitHub Copilot

Data: 2026-03-20