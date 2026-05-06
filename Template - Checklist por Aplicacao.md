# Template - Checklist por Aplicacao

## 1. Identificacao da aplicacao

- Nome da aplicacao:
- Repositorio GitHub:
- Owner tecnico:
- Owner funcional:
- Linguagem:
- Framework:

---

## 2. Ambientes e dominios

- DEV:
- HOM:
- VALIDACAO:
- PROD:

---

## 3. Runtime e empacotamento

- [ ] Possui `Dockerfile`
- [ ] Possui arquivo de compose do ambiente
- [ ] Possui `.env.example`
- [ ] Possui healthcheck
- [ ] Possui smoke test
- [ ] Possui endpoint ou log de versao implantada

Observacoes:

---

## 4. Rede e exposicao

- [ ] Banco nao esta publicado externamente
- [ ] Redis ou cache nao esta publicado externamente
- [ ] Backend nao esta publicado externamente sem justificativa
- [ ] Servico exposto no host usa `127.0.0.1`
- [ ] NGINX configurado como entrada publica unica
- [ ] UFW revisado

Observacoes:

---

## 5. Dados e dependencias

- [ ] Banco identificado
- [ ] Politica de backup definida
- [ ] Variaveis de ambiente mapeadas
- [ ] Integracoes externas mapeadas
- [ ] Credenciais fora do repositorio

Observacoes:

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

---

## 7. Promocao e rollback

- [ ] Estrategia de branches definida
- [ ] Estrategia de tags definida
- [ ] Versao implantada fica registrada
- [ ] Rollback por tag anterior documentado
- [ ] Procedimento de incidente documentado

Observacoes:

---

## 8. Criterios de pronto

- [ ] Aplicacao pronta para DEV
- [ ] Aplicacao pronta para HOM
- [ ] Aplicacao pronta para VALIDACAO
- [ ] Aplicacao pronta para PROD

Responsavel pela avaliacao:

Data: