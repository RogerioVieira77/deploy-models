# Plano Tecnico - Implementacao da Esteira

## 1. Objetivo

Transformar a análise do servidor e a arquitetura alvo em um plano técnico executável para implantação da esteira com GitHub Actions, padrão de deploy e checklist por aplicação.

Este plano prioriza:

- padronização do runtime
- redução de deploy manual
- promoção segura entre ambientes
- rollback previsível

---

## 2. Premissas do plano

1. O GitHub será a fonte de verdade do código.
2. O deploy de HOM, VALIDACAO e PROD promoverá a mesma tag.
3. As aplicações devem convergir para padrão de containerização consistente.
4. O servidor de homologação deve publicar aplicações via `127.0.0.1 + NGINX`.
5. O pipeline deve gerar evidência de versão, executor e resultado por ambiente.

---

## 3. Fases de implementação

## Fase 1 - Higienização do runtime atual

Objetivo:

- remover divergências mais perigosas antes de automatizar

Entregas:

- revisar `docker compose` das aplicações existentes
- remover exposição indevida de banco, Redis e backend
- convergir para `127.0.0.1` quando houver publicação no host
- revisar vhosts NGINX
- revisar regras UFW

Saída esperada:

- servidor de homologação padronizado

## Fase 2 - Padronização por repositório

Objetivo:

- garantir que cada app esteja pronta para CI/CD

Entregas mínimas por repositório:

- `Dockerfile`
- compose de homologação padronizado
- `.env.example`
- healthcheck
- smoke test simples
- README operacional por ambiente

Saída esperada:

- cada aplicação apta a entrar na pipeline

## Fase 3 - CI base no GitHub Actions

Objetivo:

- construir e validar as aplicações automaticamente

Entregas:

- workflow de lint
- workflow de testes
- workflow de build de imagem
- push de imagem para registry

Saída esperada:

- artefato versionado e auditável por commit e por tag

## Fase 4 - CD para DEV e HOM

Objetivo:

- automatizar deploy em DEV e HOM

Entregas:

- deploy automático para DEV a partir de `develop`
- deploy para HOM a partir de tag
- validação automática pós-deploy
- registro da versão implantada

## Fase 5 - Promoção para VALIDACAO e PROD

Objetivo:

- adicionar gates manuais e promoção segura

Entregas:

- GitHub Environments
- aprovação manual
- rollback por tag anterior
- checklist de promoção

## Fase 6 - Observabilidade e operação

Objetivo:

- sustentar a esteira em regime operacional

Entregas:

- monitoramento básico
- alertas
- runbooks
- evidência pós-deploy

---

## 4. Estrutura técnica recomendada no GitHub Actions

Cada repositório deve evoluir para conter, no mínimo:

```text
.github/workflows/
  ci.yml
  build-image.yml
  deploy-dev.yml
  deploy-hom.yml
  deploy-validacao.yml
  deploy-prod.yml
```

Descrição sugerida:

- `ci.yml`: lint, testes, validações rápidas
- `build-image.yml`: build e push da imagem para registry
- `deploy-dev.yml`: deploy automático em DEV por merge em `develop`
- `deploy-hom.yml`: deploy em HOM por tag ou release candidate
- `deploy-validacao.yml`: promoção manual da mesma tag
- `deploy-prod.yml`: promoção manual final da mesma tag

---

## 5. Fluxo recomendado de GitHub Actions

## 5.1 CI

Trigger:

- pull request para `develop`
- pull request para `main`
- push em `develop`
- push em `main`

Etapas:

1. checkout
2. setup da linguagem
3. cache de dependências
4. lint
5. testes
6. build local

## 5.2 Build de imagem

Trigger:

- push em `main`
- criação de tag `v*`

Etapas:

1. checkout
2. login no registry
3. docker build
4. tag com SHA curto
5. tag com versão semântica ou data
6. push

Tags recomendadas da imagem:

- `ghcr.io/<org>/<repo>:sha-<shortsha>`
- `ghcr.io/<org>/<repo>:v2026.03.20.1`
- opcionalmente `:main`

## 5.3 Deploy DEV

Trigger:

- merge em `develop`

Etapas:

1. resolver versão da imagem
2. conectar ao ambiente DEV
3. atualizar `.env` ou compose override
4. subir containers
5. executar smoke test
6. publicar resumo do deploy

## 5.4 Deploy HOM

Trigger:

- tag `v*`

Etapas:

1. validar existência da imagem da tag
2. aprovação opcional do responsável técnico
3. deploy remoto
4. smoke test
5. registrar evidência de homologação

## 5.5 Deploy VALIDACAO

Trigger:

- workflow dispatch manual selecionando a tag já homologada

Etapas:

1. validar que a tag já passou em HOM
2. aprovação manual
3. deploy remoto
4. smoke test
5. liberar para validação do usuário

## 5.6 Deploy PROD

Trigger:

- workflow dispatch manual para tag já validada

Etapas:

1. confirmar backup ou snapshot quando aplicável
2. aprovação manual
3. deploy da mesma tag
4. smoke test
5. monitoramento pós-implantação

---

## 6. Padrão técnico de deploy remoto

Existem três modelos possíveis.

## 6.1 SSH remoto com runner GitHub hospedado

Vantagens:

- simples de iniciar
- baixo esforço inicial

Desvantagens:

- requer gestão cuidadosa de chaves e comandos remotos

## 6.2 Runner self-hosted por ambiente

Vantagens:

- melhor controle local
- reduz dependência de acesso SSH externo por workflow

Desvantagens:

- aumenta responsabilidade operacional do runner

## 6.3 Pull-based deploy no host

Vantagens:

- host busca a versão aprovada

Desvantagens:

- maior complexidade de orquestração inicial

Recomendação pragmática para começar:

- iniciar com SSH remoto bem controlado
- evoluir para runner self-hosted se o volume de deploy crescer

---

## 7. Padrão operacional do deploy por aplicação

Cada aplicação deve expor no mínimo estes componentes operacionais:

1. arquivo de compose do ambiente
2. arquivo de env do ambiente
3. healthcheck
4. smoke test
5. comando de rollback
6. versão implantada visível em logs ou endpoint

Modelo de diretório recomendado no servidor:

```text
/opt/unicomunitaria/docker/<app>/
  compose/
  releases/
  current/
  .env
```

Modelo de estratégia simples:

- `current` aponta para a release ativa
- deploy atualiza a referência
- rollback reposiciona para a release anterior

Se preferir manter abordagem por compose direto no repositório:

- ainda assim registrar tag implantada
- ainda assim fixar imagem por versão

---

## 8. Exemplo de padrão de GitHub Environments

Configurar no GitHub:

- `dev`
- `hom`
- `validacao`
- `prod`

Em cada environment:

- secrets próprios
- variáveis próprias
- approvals quando necessário
- proteção contra deploy indevido

Secrets típicos por ambiente:

- `SSH_HOST`
- `SSH_USER`
- `SSH_PRIVATE_KEY`
- `APP_DOMAIN`
- `APP_PORT`
- `APP_ENV_FILE`
- `REGISTRY_USERNAME`
- `REGISTRY_TOKEN`

---

## 9. Padrão de checklist por aplicação

Cada aplicação deve preencher esta ficha antes de entrar na esteira.

## 9.1 Identificação

- nome da aplicação
- repositório GitHub
- branch principal
- domínio por ambiente
- owner técnico
- owner funcional

## 9.2 Runtime

- linguagem
- framework
- Dockerfile existente
- compose existente
- healthcheck existente
- logs acessíveis

## 9.3 Dependências

- banco
- cache
- filas
- integrações externas
- variáveis obrigatórias

## 9.4 Segurança

- secrets fora do repositório
- portas revisadas
- NGINX revisado
- UFW revisado
- usuários e permissões revisados

## 9.5 Pipeline

- lint implementado
- testes implementados
- build de imagem implementado
- deploy DEV implementado
- deploy HOM implementado
- smoke test implementado
- rollback documentado

---

## 10. Backlog técnico priorizado

## Prioridade alta

1. Corrigir as aplicações que ainda publicam serviços em `0.0.0.0`.
2. Definir padrão oficial de compose para homologação.
3. Criar template de workflow GitHub Actions reutilizável.
4. Definir registry de imagens.
5. Definir padrão de versionamento por tag.

## Prioridade média

1. Criar runbooks de deploy e rollback.
2. Criar script padronizado de smoke test.
3. Adicionar endpoint de versão nas aplicações.
4. Adicionar observabilidade mínima.

## Prioridade baixa

1. Provisionar host com automação versionada.
2. Evoluir para runner self-hosted por ambiente.
3. Adicionar métricas e alertas mais completos.

---

## 11. Entregáveis concretos sugeridos

### No repositório de documentação

- modelo unificado de deploy em HOM
- arquitetura alvo da esteira
- plano técnico de implementação
- checklist padrão por aplicação
- template operacional reutilizável por aplicação

### Em cada aplicação

- `.github/workflows/ci.yml`
- `.github/workflows/build-image.yml`
- `.github/workflows/deploy-dev.yml`
- `.github/workflows/deploy-hom.yml`
- arquivo compose padronizado
- `.env.example`
- smoke test

### No servidor

- diretório padronizado por aplicação
- NGINX revisado
- UFW revisado
- credenciais de deploy definidas

---

## 12. Definição de pronto para uma aplicação entrar na esteira

Uma aplicação só deve entrar no pipeline completo quando cumprir todos os itens abaixo:

- [ ] código no GitHub
- [ ] branch strategy definida
- [ ] Dockerfile válido
- [ ] compose do ambiente válido
- [ ] healthcheck funcional
- [ ] smoke test funcional
- [ ] sem banco ou cache publicado externamente
- [ ] NGINX padronizado
- [ ] secrets fora do código
- [ ] rollback documentado
- [ ] responsável técnico definido
- [ ] responsável funcional definido

---

## 13. Sequência recomendada de execução real

1. Padronizar `sistema-de-enderecos`, `polidrama` e `sistema-de-condominios` para o modelo seguro de homologação.
2. Escolher uma aplicação piloto.
3. Implantar CI e build de imagem nessa aplicação piloto.
4. Implantar deploy automático para HOM nessa aplicação piloto.
5. Validar rollback.
6. Transformar o workflow em template reutilizável.
7. Replicar nas demais aplicações.
8. Só depois abrir VALIDACAO e PROD na esteira completa.

---

## 14. Resultado esperado ao final do plano

Ao final da implementação:

- o deploy em homologação deixa de depender de operação artesanal
- cada ambiente passa a receber versões rastreáveis
- a mesma release pode ser promovida com segurança entre HOM, VALIDACAO e PROD
- rollback fica operacionalmente simples
- a produção fica mais protegida contra mudanças não controladas