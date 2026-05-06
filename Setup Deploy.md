# Setup Deploy

## Objetivo

Configurar no GitHub do repositório piloto o environment `hom` e os secrets necessários para viabilizar o workflow [deploy-hom.yml](/opt/unicomunitaria/docker/sistema-de-enderecos/.github/workflows/deploy-hom.yml).

## O que precisa existir no GitHub

1. Environment: `hom`
2. Secret: `HOM_SSH_HOST`
3. Secret: `HOM_SSH_USER`
4. Secret: `HOM_SSH_PRIVATE_KEY`

## O que cada secret deve conter

### `HOM_SSH_HOST`

Valor: IP ou hostname do servidor de homologação.

Valor direto para o cenário atual:

```text
191.101.234.42
```

### `HOM_SSH_USER`

Valor: usuário SSH que o GitHub Actions vai usar para entrar no servidor.

Recomendação:

- idealmente usar um usuário dedicado de deploy, por exemplo `deploy` ou `github-actions`
- esse usuário precisa ter acesso ao diretório da aplicação e permissão para executar Docker

### `HOM_SSH_PRIVATE_KEY`

Valor: chave privada SSH completa correspondente à chave pública autorizada no servidor para o usuário acima.

Formato esperado:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

## Passo a passo no GitHub

1. Abrir o repositório piloto `sistema-de-enderecos` no GitHub.
2. Entrar em `Settings`.
3. No menu lateral, abrir `Environments`.
4. Clicar em `New environment`.
5. Criar o environment com o nome exato `hom`.
6. Entrar no environment `hom`.
7. Em `Environment secrets`, clicar em `Add secret`.
8. Cadastrar `HOM_SSH_HOST` com o valor `191.101.234.42`.
9. Cadastrar `HOM_SSH_USER` com o usuário SSH do deploy.
10. Cadastrar `HOM_SSH_PRIVATE_KEY` com a chave privada completa.
11. Salvar.

## Configurações recomendadas no environment `hom`

### `Environment URL`

Preencher com a URL pública de homologação da aplicação piloto, se quiser deixar o destino visível na tela de deploy.

Exemplo:

```text
https://sistema-de-enderecos.unicomunitaria.com.br
```

### `Deployment branches and tags`

Como o workflow de deploy é disparado por tag `v*`, vale restringir o uso do environment ao fluxo oficial de release, quando o plano da conta permitir essa política.

### `Required reviewers`

Se quiser gate manual antes de implantar em HOM, adicione 1 ou 2 revisores.

Isso faz o job pausar antes de liberar o uso dos secrets do environment.

## Ponto importante sobre a chave SSH

A chave pública correspondente ao `HOM_SSH_PRIVATE_KEY` precisa estar no servidor, no arquivo `~/.ssh/authorized_keys` do usuário definido em `HOM_SSH_USER`.

Sem isso, o workflow vai falhar na etapa de SSH.

## Checklist do lado do servidor para o usuário de deploy

1. O usuário consegue acessar por SSH sem senha usando a chave.
2. O usuário consegue entrar em `/opt/unicomunitaria/docker/sistema-de-enderecos`.
3. O usuário consegue executar:

```bash
git fetch --all --tags
docker compose config
make bootstrap
```

4. O usuário tem permissão para usar Docker.

Normalmente isso significa estar no grupo `docker`, ou usar outro arranjo já validado no servidor.

## Como testar depois de configurar

1. No GitHub, abrir `Actions`.
2. Selecionar o workflow `Deploy HOM`.
3. Clicar em `Run workflow`.
4. Informar no campo `ref` uma tag existente, por exemplo `v1.0.0`.
5. Executar.
6. Verificar se passa pelas etapas:

- `Start SSH agent`
- `Add target host to known_hosts`
- `Deploy selected ref to HOM`

## O que esse workflow faz de fato

1. Faz checkout do repositório.
2. Carrega a chave privada do secret.
3. Faz `ssh-keyscan` do host.
4. Conecta no servidor por SSH.
5. Vai para `/opt/unicomunitaria/docker/sistema-de-enderecos`.
6. Faz `git fetch --all --tags`.
7. Faz `git checkout -f` da tag ou ref escolhida.
8. Valida o `docker compose`.
9. Executa `make bootstrap`.
10. Faz smoke test local:

- `/health`
- teste do endpoint de cadastro

## Resumo prático das telas do GitHub

1. `Repository`
2. `Settings`
3. `Environments`
4. `hom`
5. `Environment secrets`
6. Criar:

- `HOM_SSH_HOST=191.101.234.42`
- `HOM_SSH_USER=<usuario_ssh>`
- `HOM_SSH_PRIVATE_KEY=<chave_privada_completa>`