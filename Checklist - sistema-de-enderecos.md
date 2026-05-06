# Checklist - sistema-de-enderecos

## 1. Identificacao da aplicacao

- Nome da aplicacao: sistema-de-enderecos
- Repositorio GitHub: `git@github.com:RogerioVieira77/sistema-de-enderecos.git`
- Owner tecnico: a definir
- Owner funcional: a definir
- Linguagem: Python
- Framework: FastAPI + PostGIS

---

## 2. Ambientes e dominios

- DEV: a definir
- HOM: `sistema-de-enderecos.unicomunitaria.com.br`
- VALIDACAO: a definir
- PROD: a definir

---

## 3. Runtime e empacotamento

- [x] Possui `Dockerfile`
- [x] Possui arquivo de compose do ambiente
- [x] Possui `.env.example`
- [x] Possui healthcheck
- [x] Possui smoke test
- [ ] Possui endpoint ou log de versao implantada

Observacoes:

- `docker-compose.yml` foi ajustado para bind local em `127.0.0.1:8000`.
- Healthcheck identificado em `/health`.
- Smoke test existente em `scripts/test_cadastro_endpoint.sh`.
- Repositorio estava limpo no momento da analise.

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
- Este repositorio e o melhor candidato a aplicacao piloto da esteira.

---

## 5. Dados e dependencias

- [x] Banco identificado
- [ ] Politica de backup definida
- [x] Variaveis de ambiente mapeadas
- [x] Integracoes externas mapeadas
- [ ] Credenciais fora do repositorio

Observacoes:

- Banco PostGIS interno via servico `db`.
- Dependencia operacional relevante: carga inicial CSV e bootstrap de schema.
- Ainda precisa formalizar secrets de deploy por ambiente.

---

## 6. Pipeline GitHub Actions

- [x] Workflow de CI implementado
- [ ] Workflow de build de imagem implementado
- [ ] Workflow de deploy DEV implementado
- [x] Workflow de deploy HOM implementado
- [ ] Workflow de deploy VALIDACAO definido
- [ ] Workflow de deploy PROD definido
- [ ] GitHub Environments configurados
- [ ] Secrets por ambiente configurados

Observacoes:

- Workflow `ci.yml` criado com bootstrap, healthcheck e validacao de contrato.
- Workflow `deploy-hom.yml` criado com deploy por tag ou `workflow_dispatch`, dependente de secrets do GitHub Environment `hom`.

---

## 7. Promocao e rollback

- [ ] Estrategia de branches definida
- [ ] Estrategia de tags definida
- [ ] Versao implantada fica registrada
- [ ] Rollback por tag anterior documentado
- [ ] Procedimento de incidente documentado

Observacoes:

- O workflow de HOM aceita tag ou referencia manual, mas a politica formal de promocao ainda precisa ser fechada no GitHub.

---

## 8. Criterios de pronto

- [x] Aplicacao pronta para DEV
- [x] Aplicacao pronta para HOM
- [ ] Aplicacao pronta para VALIDACAO
- [ ] Aplicacao pronta para PROD

Responsavel pela avaliacao: GitHub Copilot

Data: 2026-03-20