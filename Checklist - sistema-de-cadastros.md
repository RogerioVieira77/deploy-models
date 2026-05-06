# Checklist - sistema-de-cadastros

## 1. Identificacao da aplicacao

- Nome da aplicacao: sistema-de-cadastros
- Repositorio GitHub: `git@github.com:RogerioVieira77/sistema-de-cadastros.git`
- Owner tecnico: a definir
- Owner funcional: a definir
- Linguagem: Python + JavaScript
- Framework: Flask + Nginx + PostgreSQL

---

## 2. Ambientes e dominios

- DEV: a definir
- HOM: `sistema-de-cadastro.unicomunitaria.com.br`
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

- `docker-compose.yml` ja esta alinhado ao modelo seguro: frontend em `127.0.0.1:8081`, backend e banco sem publicacao externa.
- Endpoint de health identificado em `/health`.
- Nao foram encontrados testes automatizados no repositorio.

---

## 4. Rede e exposicao

- [x] Banco nao esta publicado externamente
- [x] Redis ou cache nao esta publicado externamente
- [x] Backend nao esta publicado externamente sem justificativa
- [x] Servico exposto no host usa `127.0.0.1`
- [x] NGINX configurado como entrada publica unica
- [x] UFW revisado

Observacoes:

- Vhost ativo em homologacao identificado no host.
- Aplicacao atualmente e a melhor referencia de runtime seguro entre os projetos inspecionados.

---

## 5. Dados e dependencias

- [x] Banco identificado
- [ ] Politica de backup definida
- [x] Variaveis de ambiente mapeadas
- [x] Integracoes externas mapeadas
- [ ] Credenciais fora do repositorio

Observacoes:

- Banco PostgreSQL interno via servico `db`.
- Integracao externa principal: API de enderecos.
- Ainda depende de `.env` local por ambiente; governanca de secrets ainda precisa ser formalizada.

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

- Rollback tecnico e possivel por Git + compose, mas nao esta padronizado por tag/versionamento ainda.

---

## 8. Criterios de pronto

- [ ] Aplicacao pronta para DEV
- [x] Aplicacao pronta para HOM
- [ ] Aplicacao pronta para VALIDACAO
- [ ] Aplicacao pronta para PROD

Responsavel pela avaliacao: GitHub Copilot

Data: 2026-03-20