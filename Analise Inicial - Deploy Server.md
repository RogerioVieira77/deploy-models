# Análise Inicial - Servidor de Deploy e Homologação

## 1. Objetivo desta análise

Este documento consolida a leitura da documentação existente com o estado real do servidor atualmente em operação, para servir de base ao desenho da esteira de deploy entre Desenvolvimento, GitHub, Homologação, Validação e Produção.

O foco aqui não é definir ainda a pipeline final, mas registrar:

- o que já está funcionando
- onde existem discrepâncias entre documentação e ambiente real
- quais riscos operacionais e de segurança existem hoje
- quais pré-condições precisam ser resolvidas antes de automatizar a esteira

---

## 2. Resumo executivo

O servidor atual já atende parcialmente ao papel de ambiente intermediário de homologação.

Pontos positivos já confirmados:

- Docker está sendo usado como padrão de isolamento das aplicações.
- NGINX no host centraliza a publicação HTTP/HTTPS nas portas 80 e 443.
- UFW está ativo e bloqueia portas altas expostas por containers.
- Há integração com GitHub nos repositórios implantados.
- Existe pelo menos um padrão funcional de publicação segura via `127.0.0.1 + NGINX`, já aplicado no sistema de cadastro.

Pontos críticos identificados:

- A documentação está redundante, fragmentada e parcialmente desatualizada.
- O padrão operacional ainda não é uniforme entre as aplicações.
- Algumas stacks publicam backend, frontend, banco e Redis em `0.0.0.0`, dependendo do UFW para contenção.
- Não foram encontrados workflows versionados em `.github/workflows`, o que indica ausência de CI/CD padronizado no nível dos repositórios presentes no servidor.
- O fluxo alvo informado mistura promoção por ambiente com “servidor para GitHub”, o que precisa ser redesenhado para um modelo de promoção por branch, tag ou release.

Conclusão inicial:

O servidor está funcional como base de homologação, mas ainda não está padronizado o suficiente para suportar uma esteira confiável sem antes consolidar arquitetura, convenções de deploy, promoção de versão e observabilidade.

---

## 3. Estado real do servidor analisado

## 3.1 Sistema operacional e host

- Hostname: `srv1312297`
- Sistema operacional: `Ubuntu 24.04.4 LTS`
- Kernel: `6.8.0-100-generic`
- Virtualização: `KVM`

## 3.2 Endereçamento observado no servidor

IP público identificado no host:

- IPv4: `191.101.234.42`

Observação importante:

- No objetivo informado, o IP de DEV e HOM foi descrito como `82.25.75.88`.
- Isso diverge do IP efetivamente observado no servidor analisado.
- Antes de desenhar a esteira, é necessário confirmar se:
  - o IP informado está incorreto,
  - existe NAT ou balanceamento à frente do host,
  - ou o servidor analisado não é exatamente o mesmo ativo no fluxo desejado.

## 3.3 Serviços de entrada no host

Portas expostas no host:

- `22/tcp` - SSH
- `80/tcp` - NGINX HTTP
- `443/tcp` - NGINX HTTPS

Portas adicionais publicadas por containers:

- `3000/tcp`
- `8000/tcp`
- `8081/tcp` em `127.0.0.1`
- `8091/tcp`
- `8092/tcp`
- `5434/tcp`
- `6380/tcp`
- `15432/tcp`

## 3.4 Firewall atual

O UFW está ativo com o seguinte comportamento observado:

- Permite `Nginx Full`
- Permite `OpenSSH`
- Bloqueia externamente:
  - `8000/tcp`
  - `8091/tcp`
  - `8092/tcp`
  - `15432/tcp`
  - `8081/tcp`

Leitura arquitetural:

- O host está protegido por firewall e isso reduz exposição indevida.
- Porém o desenho atual ainda depende do firewall para bloquear portas que idealmente nem deveriam ser publicadas em `0.0.0.0`.

---

## 4. Aplicações e topologia atualmente implantadas

Diretórios encontrados em `/opt/unicomunitaria/docker`:

- `sistema-de-cadastros`
- `sistema-de-enderecos`
- `polidrama`
- `sistema-de-condominios`
- `app001`
- `app002`
- `app003`

## 4.1 Vhosts NGINX ativos

Ativos em `sites-enabled`:

- `sistema-de-cadastro.hml.conf`
- `sistema-de-enderecos.hml.conf`

Disponíveis em `sites-available`:

- `default`
- `polidrama`
- `sistema-de-cadastro.hml.conf`
- `sistema-de-enderecos.hml.conf`

Leitura:

- Existem apenas dois vhosts homologados explicitamente ativos.
- `polidrama` existe em `sites-available`, mas não está habilitado no conjunto inspecionado.
- Nem todas as aplicações em execução parecem seguir a mesma estratégia de publicação via NGINX.

## 4.2 Repositórios Git encontrados

Todos os principais projetos analisados possuem `origin` configurado para GitHub e branch atual `main`:

- `RogerioVieira77/sistema-de-cadastros`
- `RogerioVieira77/sistema-de-enderecos`
- `RogerioVieira77/polidrama`
- `RogerioVieira77/sistema-de-condominios`

Leitura:

- O servidor já está acoplado ao GitHub como origem de código.
- Isso é suficiente para operação manual assistida.
- Ainda não é suficiente para promoção automatizada entre ambientes.

## 4.3 Ausência de workflows CI/CD detectáveis

Na varredura feita no conteúdo presente em `/opt/unicomunitaria/docker`, não foram encontrados arquivos em `.github/workflows`.

Leitura:

- Não há evidência local de pipeline padronizada em GitHub Actions.
- O deploy atual aparenta ser majoritariamente manual ou assistido por comandos locais.

---

## 5. Comparativo entre padrão documentado e estado real

## 5.1 Padrão documentado mais seguro

O documento principal propõe como padrão:

- app publicada apenas em `127.0.0.1`
- NGINX como único ponto de exposição pública
- bancos e backends sem publicação externa
- proteção complementar via UFW

Esse padrão é tecnicamente correto e é o mais adequado para homologação multiaplicação no mesmo host.

## 5.2 Estado real por aplicação

### Sistema de Cadastros

`docker-compose.yml` segue um padrão mais aderente ao desejado:

- banco sem `ports`
- backend sem `ports`
- frontend exposto apenas em `127.0.0.1:8081:80`
- NGINX ativo apontando para `127.0.0.1:8081`

Avaliação:

- É a referência mais madura de padrão para o servidor de homologação.

### Sistema de Endereços

`docker-compose.yml` publica a API em:

- `8000:8000`

O UFW bloqueia a porta externamente, mas o serviço segue publicado em `0.0.0.0`.

Avaliação:

- Funciona, mas não segue o padrão mais seguro descrito na documentação.
- Idealmente deveria publicar em `127.0.0.1:8000:8000` ou nem publicar porta se houver rede interna dedicada com NGINX apontando para upstream adequado.

### Polidrama

`docker-compose.yml` publica:

- backend em `8091:8000`
- frontend em `8092:80`
- banco em `15432:5432`

O UFW bloqueia essas portas externamente.

Avaliação:

- O modelo depende fortemente de firewall.
- Banco publicado no host, ainda que bloqueado externamente, aumenta superfície operacional e risco de erro humano.
- É um candidato claro à padronização antes da automação.

### Sistema de Condomínios

Foi encontrado `docker-compose.staging.yml`, com publicação em:

- frontend `3000:3000`
- backend `8000:8000`
- postgres `5434:5432`
- redis `6380:6379`

Avaliação:

- O perfil é claramente de ambiente de staging/desenvolvimento, não de homologação endurecida.
- Uso de `--reload` no backend reforça esse ponto.
- A stack precisa ser adaptada antes de integrar uma esteira de promoção segura.

---

## 6. Análise por perfil solicitado

## 6.1 Software Architect

Achados:

- Já existe um padrão arquitetural implícito: aplicações containerizadas, entrada única via NGINX e segregação por subdomínio.
- O padrão ainda não virou convenção obrigatória entre todos os sistemas.
- O fluxo desejado de promoção entre ambientes ainda não está modelado em termos de branches, tags, artefatos, critérios de aprovação e rollback.

Leitura arquitetural:

- O ideal é abandonar a lógica “servidor empurra para GitHub” e adotar promoção de artefato ou de commit entre ambientes.
- O GitHub deve ser a fonte de verdade, e os servidores devem apenas consumir versões aprovadas.

Recomendação inicial:

- Definir um fluxo único de promoção, por exemplo:
  - `develop` -> ambiente DEV
  - `main` ou `release/*` -> ambiente HOM
  - tag aprovada -> ambiente VALIDACAO
  - release assinada/aprovada -> PRODUCAO

## 6.2 Security Architect

Achados:

- NGINX com TLS wildcard já está operacional.
- UFW está corretamente ativado e reduz exposição.
- Ainda existem portas sensíveis publicadas no host por algumas aplicações.
- Bancos e Redis não deveriam depender de regra de firewall como controle principal.
- Ainda não foi verificado uso de secrets centralizados, rotação de credenciais ou segregação forte entre ambientes.

Riscos:

- Exposição acidental futura ao alterar UFW.
- Diferença entre configuração documentada e configuração real.
- Possível reutilização de credenciais entre ambientes.

Recomendação inicial:

- Tornar obrigatório o padrão “bind local ou sem bind externo”.
- Separar arquivos `.env` por ambiente com política clara de gestão.
- Mapear autenticação de deploy e acesso administrativo.

## 6.3 Data Architect

Achados:

- Há bancos PostgreSQL/PostGIS em múltiplas stacks.
- Pelo menos um Redis em staging.
- Não há ainda registro unificado de estratégia de backup, retenção, restauração e mascaramento de dados para homologação.

Riscos:

- Homologação usar dados sensíveis sem política definida.
- Falta de padronização de backup antes de deploy ou migração.

Recomendação inicial:

- Definir política de dados por ambiente:
  - DEV: dados sintéticos
  - HOM: dados mascarados ou sintéticos representativos
  - VALIDACAO: política por aplicação
  - PROD: backup/versionamento formal antes de cada mudança estrutural

## 6.4 DevOps Engineer

Achados:

- Repositórios estão no GitHub.
- Não há evidência de GitHub Actions versionado nos projetos inspecionados.
- Deploy atual depende de execução manual de `docker compose`, ajustes em NGINX e firewall.
- O servidor de homologação já tem base adequada para receber deploy automatizado.

Leitura DevOps:

- A base existe, mas a esteira ainda não existe como produto operacional.
- Falta padronizar:
  - build
  - testes
  - versionamento de imagem
  - promoção entre ambientes
  - rollback
  - smoke test pós-deploy

## 6.5 Cloud Architect

Achados:

- O ambiente atual é essencialmente um host único com múltiplas aplicações dockerizadas.
- Não há evidência de IaC para o host, NGINX, UFW, certificados, diretórios e convenções operacionais.

Riscos:

- Dependência de configuração manual no servidor.
- Dificuldade de reconstrução rápida em caso de incidente.

Recomendação inicial:

- Tratar este servidor como infraestrutura reproduzível.
- Codificar baseline de sistema com Ansible, Terraform + cloud-init, ou ao menos shell provisionado versionado.

## 6.6 SRE

Achados:

- O ambiente roda múltiplas aplicações no mesmo host.
- Não foi identificada estratégia formal de health checks externos, SLO, rollback automatizado ou dreno controlado.

Riscos:

- Falha operacional em uma stack pode impactar troubleshooting das demais.
- Sem telemetria mínima, a validação pós-deploy tende a ser manual e subjetiva.

Recomendação inicial:

- Padronizar health endpoint, readiness e smoke test por aplicação.
- Definir política de restart, retenção de logs, uso de volumes e rollback de imagem.

## 6.7 Monitoring

Achados:

- Não foram observados elementos explícitos de Prometheus, Grafana, OpenTelemetry ou alertas centralizados na inspeção inicial.

Recomendação inicial:

- Implantar observabilidade mínima antes da esteira final:
  - métricas do host
  - métricas de containers
  - uptime HTTP/HTTPS por domínio
  - alertas para indisponibilidade e falha de deploy

## 6.8 Incident Response

Achados:

- A operação atual parece dependente de acesso direto ao host para diagnóstico.
- Não há evidência de playbooks de rollback ou runbooks formais centralizados.

Recomendação inicial:

- Criar runbooks curtos para:
  - rollback de deploy
  - restauração de banco
  - falha de NGINX
  - falha de container
  - renovação/validação de certificado

## 6.9 Product Improvement

Achados:

- O objetivo do servidor está alinhado com proteção da produção e validação progressiva.
- Ainda não há critérios formais de aceite entre HOM, VALIDACAO e PROD.

Recomendação inicial:

- Definir gates objetivos por ambiente:
  - sucesso de testes automatizados
  - smoke test
  - checklist funcional
  - aprovação manual para promoção

---

## 7. Principais discrepâncias documentais encontradas

1. A documentação mistura modelo geral, modelo rápido, portas em uso e caso específico de aplicação, com trechos repetidos.

2. O documento `PORTAS_EM_USO` está desatualizado em relação ao estado atual do host. Hoje também existem portas `3000`, `5434` e `6380` publicadas.

3. O padrão documentado recomenda faixa `8081-8099`, porém o servidor atual usa também `8000`, `3000`, `5434`, `6380` e `15432`.

4. Nem todas as aplicações seguem o padrão de bind local em `127.0.0.1`.

5. A documentação atual descreve muito bem o deploy manual assistido, mas ainda não descreve promoção entre ambientes, versionamento de release, rollback ou critérios de validação.

6. O IP informado no objetivo do fluxo diverge do IP público observado no servidor inspecionado.

---

## 8. Diagnóstico inicial de maturidade

Maturidade atual por domínio:

- Containerização: média
- Proxy reverso e HTTPS: média/boa
- Hardening básico de host: média
- Padronização entre aplicações: baixa
- CI/CD versionado: baixa
- Observabilidade: baixa
- Governança de promoção entre ambientes: baixa
- Infraestrutura reproduzível: baixa

---

## 9. Recomendação objetiva para a próxima etapa

Antes de montar a esteira completa, a próxima fase deveria seguir esta ordem:

1. Consolidar a documentação em um único padrão operacional por ambiente.

2. Normalizar as stacks para o padrão de homologação:
   - sem banco publicado externamente
   - sem backend publicado externamente quando não necessário
   - frontend ou upstream apenas em `127.0.0.1` quando publicado no host
   - NGINX como ponto único de entrada pública

3. Definir a estratégia oficial de promoção entre ambientes:
   - branch
   - tag
   - release
   - imagem Docker versionada

4. Definir o papel de cada ambiente:
   - DEV
   - HOM
   - VALIDACAO
   - PROD

5. Criar um padrão mínimo de pipeline GitHub Actions:
   - build
   - testes
   - lint
   - geração/versionamento de imagem
   - deploy automatizado por ambiente
   - smoke test
   - rollback simples

6. Implantar observabilidade mínima e runbooks operacionais.

---

## 10. Conclusão

O servidor analisado já cumpre parcialmente a função de ambiente intermediário de homologação e já possui base técnica suficiente para evoluir para uma esteira de deploy.

Porém, a automação neste momento não deve começar diretamente pela pipeline. O maior ganho agora está em padronizar o runtime real das aplicações e consolidar a documentação, porque hoje existe diferença entre o modelo desejado e o modo como parte das aplicações realmente está publicada.

Se essa normalização vier antes da esteira, o pipeline ficará mais simples, mais seguro e mais fácil de replicar por aplicação.