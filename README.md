# Repository Sample

Repositório de exemplo da organização [Tech-Preta](https://github.com/Tech-Preta) para demonstrar uma estrutura de repositório no GitHub. O código atual não contém aplicação, pacote ou serviço executável: o conteúdo é documentação de comunidade, configuração de revisão e workflows de GitHub Actions.

No GitHub, o repositório [Tech-Preta/repository-sample](https://github.com/Tech-Preta/repository-sample) está público, com branch padrão `main`, licença GPL-3.0, tópico `sample` e a opção de template ativada (`isTemplate: true`). A descrição do repositório está vazia. A homepage configurada é <https://nataliagranato.xyz>.

## Estrutura do repositório

```text
.
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
├── .coderabbit.yaml
├── .editorconfig
├── .gitignore
├── .pre-commit-config.yaml
└── .github/
    ├── CODEOWNERS
    ├── FUNDING.yml
    ├── dependabot.yml
    ├── ISSUE_TEMPLATE/
    │   └── BUG_REPORT.md
    ├──  PULL_REQUEST_TEMPLATE/          # o nome do diretório começa com espaço
    │   └── PULL_REQUEST_TEMPLATE.md
    └── workflows/
        ├── dependabot.yml               # manifesto do Dependabot, sem `on`/`jobs`
        ├── dependency-review.yml
        ├── issue.yml
        └── trivy.yml
```

Não há `go.mod`, `package.json`, `Dockerfile`, manifesto Kubernetes, Helm chart nem diretório de testes. O `.gitignore` e o `.editorconfig` incluem regras para Go, embora nenhum arquivo `.go` exista no repositório.

## Pré-requisitos

Para usar este repositório como template, basta uma conta no GitHub. Não há runtime, gerenciador de pacotes ou versão de linguagem declarados.

Para clonar e propor mudanças neste repositório:

- [Git](https://git-scm.com/)
- Branch de comparação usada em `CONTRIBUTING.md`: `main`

Os hooks de `.pre-commit-config.yaml` dependem da ferramenta [pre-commit](https://pre-commit.com/). Este repositório não fixa a versão dessa ferramenta nem inclui script de instalação.

Os workflows rodam em `ubuntu-latest` no GitHub Actions. Dois deles leem o segredo `USER_TOKEN`. O valor não está no repositório. Não há `.env.example`. O `.gitignore` ignora `.env`, e nenhum arquivo do repositório lê variáveis desse arquivo.

## Como usar

### Criar um repositório a partir do template

1. Abra <https://github.com/Tech-Preta/repository-sample>.
2. Use **Use this template** e crie um repositório novo. Os arquivos deste template são a cópia que o GitHub oferece nesse fluxo. Segredos de Actions e regras de branch protection não estão versionados aqui; cada repositório de destino configura os seus.

### Clonar este repositório

```sh
git clone https://github.com/Tech-Preta/repository-sample.git
cd repository-sample
```

`CONTRIBUTING.md` ainda mostra `https://github.com/Tech-Preta/repository_sample` (com sublinhado). A URL remota deste repositório usa hífen: `repository-sample`.

### Executar os hooks locais

Com o [pre-commit](https://pre-commit.com/) instalado na máquina:

```sh
pre-commit install
pre-commit run --all-files
```

Esses comandos são a interface da ferramenta referenciada pelo arquivo de configuração. Nenhum workflow deste repositório executa o pre-commit.

Hooks declarados:

| Repositório do hook | Revisão | Hook |
| --- | --- | --- |
| `https://github.com/gitleaks/gitleaks` | `v8.16.3` | `gitleaks` |
| `https://github.com/pre-commit/pre-commit-hooks` | `v4.4.0` | `end-of-file-fixer`, `trailing-whitespace` |

Não há `.gitleaks.toml` nem arquivo de baseline do gitleaks.

### O que a CI faz

Workflows versionados que seguem o formato do GitHub Actions (`on` e `jobs`):

| Arquivo | Nome | Disparo | Permissões | O que o arquivo faz |
| --- | --- | --- | --- | --- |
| `.github/workflows/dependency-review.yml` | Dependency Review | `pull_request` | `contents: read` | Faz checkout e roda `actions/dependency-review-action` |
| `.github/workflows/issue.yml` | Resposta Automática a Issues | `issues` do tipo `opened` | `contents: read`, `issues: write` | Publica um comentário na issue |
| `.github/workflows/trivy.yml` | Generate SBOM | `push` na branch `main` | `contents: write` | Roda Trivy no sistema de arquivos e envia o resultado no formato `github` |

`dependency-review.yml` e `issue.yml` começam com `step-security/harden-runner` e `egress-policy: audit`. `trivy.yml` não usa o harden-runner.

Actions fixadas por commit SHA, com a versão anotada no próprio YAML:

- `step-security/harden-runner@bf7454d06d71f1098171f2acdf0cd4708d7b5920` (`v2.20.0`)
- `actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0` (`v7.0.0`), só em `dependency-review.yml`
- `actions/dependency-review-action@a1d282b36b6f3519aa1f3fc636f609c47dddb294` (`v5.0.0`)
- `derekprior/add-autoresponse@500f8788d667c31d43a4469f8a3ec3b491470fd1` (comentário no arquivo: `master`)

`trivy.yml` referencia tags, sem SHA: `actions/checkout@v7` e `aquasecurity/trivy-action@v0.36.0`.

O comentário no topo de `dependency-review.yml` diz que a action examina manifestos de dependência alterados no pull request, aponta versões com vulnerabilidades conhecidas e bloqueia o merge se a execução estiver marcada como obrigatória. Nenhum arquivo deste repositório marca esse check como obrigatório. Também não há manifesto de dependências de aplicação, como `package.json` ou `go.mod`. O arquivo do workflow não descreve o que a action faz com os YAML de GitHub Actions.

O texto publicado por `issue.yml` agradece a pessoa que abriu a issue (login em `github.event.issue.user.login`). Quando a issue é um bug, uma pergunta sobre o produto ou uma solicitação de funcionalidade, o texto indica o e-mail `contato@nataliagranato.xyz`. A action recebe `secrets.USER_TOKEN` na variável de ambiente `GITHUB_TOKEN`.

`trivy.yml` usa `scan-type: fs`, `format: github`, `output: dependency-results.sbom.json`, `image-ref: '.'` e passa `secrets.USER_TOKEN` em `github-pat`. O nome do passo diz que o resultado é enviado ao Dependency Graph. O workflow não faz upload do JSON como artifact e não faz commit desse arquivo.

Há ainda workflows dinâmicos do GitHub, sem arquivo correspondente na árvore:

- **Dependabot Updates**, alimentado por `.github/dependabot.yml`.
- **CodeQL**, com caminho `dynamic/github-code-scanning/codeql`. Duas execuções conferidas na `main` concluíram com sucesso: a agendada [41572394029](https://github.com/Tech-Preta/repository-sample/actions/runs/41572394029) (2026-11-03) e a de push [40879210066](https://github.com/Tech-Preta/repository-sample/actions/runs/40879210066) (2026-10-29). A consulta à configuração de code scanning default setup retornou HTTP 403 para o token usado nesta documentação; linguagens, suíte de queries e agenda ficam pendentes.

`.github/workflows/dependabot.yml` repete a configuração do Dependabot (`version: 2`, ecossistema `github-actions`, diretório `/`, intervalo `daily`) e não declara `on` nem `jobs`. No GitHub Actions esse caminho aparece como workflow. As execuções listadas falham sem jobs e sem log. Exemplo na `main`: run [40879209935](https://github.com/Tech-Preta/repository-sample/actions/runs/40879209935) (2026-10-29). A configuração que o Dependabot usa é `.github/dependabot.yml`.

O histórico do Git registra a remoção de outros workflows, que não fazem parte da árvore atual: `osv-scanner.yml`, `scorecard.yml`, `checkov.yml`, `lint.yml`, `issues.yml`, `pull_request.yml`, `snyk-security.yml` e `fossa.yml`. Não há `codeql.yml` no histórico.

## Configuração

### Segredo `USER_TOKEN`

Nome do segredo de Actions referenciado no código. O valor não é documentado aqui.

| Workflow | Campo | Uso no arquivo |
| --- | --- | --- |
| `.github/workflows/issue.yml` | `env.GITHUB_TOKEN` | Token da action `derekprior/add-autoresponse` |
| `.github/workflows/trivy.yml` | `with.github-pat` | PAT passado ao `aquasecurity/trivy-action` |

Os escopos exigidos não estão descritos nos workflows. Nas execuções conferidas em 2026-10-29, a credencial foi rejeitada:

- Issues, run [40857447925](https://github.com/Tech-Preta/repository-sample/actions/runs/40857447925) (2026-10-29): o job `resposta-automatica` falhou com `Error adding autoresponse: Bad credentials`. É a única execução listada desse workflow.
- SBOM, run [40879212127](https://github.com/Tech-Preta/repository-sample/actions/runs/40879212127) (2026-10-29): o job foi marcado como sucesso e o log do upload do GitHub Dependency Snapshot contém `"message": "Bad credentials"` e `"status": "401"`.

O estado atual do segredo, depois dessas execuções, está pendente.

### Dependabot

`.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: daily
```

Só o ecossistema `github-actions` está configurado, na raiz, com checagem diária.

### CodeRabbit

`.coderabbit.yaml` define a revisão automática. Valores presentes no arquivo:

- `language: pt-BR`
- `early_access: true` e `enable_free_tier: true`
- perfil de review `chill`
- `request_changes_workflow`, `high_level_summary`, `review_status`, `poem` e `sequence_diagrams` habilitados
- `collapse_walkthrough: false`
- `abort_on_close: true`
- revisão automática habilitada, inclusive revisão incremental; `drafts: false`
- `ignore_title_keywords`, `labels`, `base_branches`, `path_filters` e `path_instructions` vazios
- `chat.auto_reply: true`
- knowledge base com `opt_out: false`, `learnings.scope: auto` e `issues.scope: auto`
- `jira.project_keys` e `linear.team_keys` vazios

O placeholder de resumo é `@coderabbitai summary` e o de título é `@coderabbitai`.

Ferramentas com `enabled: true`: shellcheck, ruff, markdownlint, github-checks (`timeout_ms: 90000`), languagetool (`level: default`, `enabled_only: false`, categorias `TYPOS`, `TYPOGRAPHY` e `CASING` desabilitadas), biome, hadolint, swiftlint, phpstan (`level: default`), golangci-lint, yamllint, gitleaks, checkov, detekt, eslint, rubocop e buf.

O Checkov também aparece em `SECURITY.md`. O workflow `checkov.yml` foi removido e não existe na árvore atual.

### EditorConfig

`.editorconfig` vale para o repositório inteiro (`root = true`):

- charset UTF-8, fim de linha LF, indentação de 4 espaços, largura de tab 4
- newline no fim do arquivo, corte de espaço no fim da linha, linha de no máximo 120 caracteres
- Markdown: sem corte de espaço no fim da linha e sem limite de comprimento
- `*.go`, `*.go2`, `go.mod` e `go.sum`: indentação com tab
- YAML: indentação de 2 espaços

### `.gitignore`

Ignora metadados do IntelliJ (`.idea`, `*.iws`, `*.iml`, `*.ipr`, `out/`), binários e artefatos de teste Go (`*.exe`, `*.dll`, `*.so`, `*.dylib`, `*.test`, `*.out`, `go.work`), `golangci-lint.xml`, arquivos locais do VS Code (`.vscode/*`, `.history/`, `.history`, `*.vsix`, `.ionide`), `.DS_Store`, `.env` e `**/__debug_bin*`. A linha `vendor/` está comentada, então esse diretório não entra na lista de ignorados.

### CODEOWNERS

`.github/CODEOWNERS` define `@nataliagranato` como proprietário de todos os caminhos (`*`).

### Financiamento

`.github/FUNDING.yml` declara:

- `github: nataliagranato`
- `ko_fi: nataliagranato`
- `buy_me_a_coffee: nataliagranato`
- `pix`: uma chave está no arquivo; o valor não é repetido neste README

### Templates de issue e de pull request

`.github/ISSUE_TEMPLATE/BUG_REPORT.md` cria o template **Bug Report**, com título prefixado por `[BUG]`, label `bug` e assignee `nataliagranato`. O corpo pede descrição, capturas de tela, ambiente, logs e issues relacionadas. As opções de ambiente são Kubernetes com manifestos, Kubernetes com Helm e Docker local, além de sistema operacional, navegador e versão da aplicação. Esses artefatos não existem neste repositório; o template apenas oferece essas opções.

O modelo de pull request está em `.github/ PULL_REQUEST_TEMPLATE/PULL_REQUEST_TEMPLATE.md`. O diretório se chama ` PULL_REQUEST_TEMPLATE`, com um espaço no início. Esse nome veio do commit que renomeou `.github/PULL_REQUEST_TEMPLATE.md` para esse caminho. O caminho documentado pelo GitHub para template automático é `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md` ou o diretório `.github/PULL_REQUEST_TEMPLATE/` sem espaço. Se o GitHub aplica o arquivo deste repositório de forma automática está pendente.

O modelo pede título, descrição, tipo de mudança (bugfix, funcionalidade, documentação, segurança, Kubernetes, Helm, Docker, GitHub Actions ou outro), checklist de estilo, testes e módulos downstream, capturas e issues relacionadas.

### Changelog e versão

`CHANGELOG.md` segue [Mantenha um Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e [Versionamento Semântico](https://semver.org/lang/pt-BR/). A única versão lançada no arquivo é `0.0.0`, datada de 2025-03-25, com o texto "Esse é apenas um exemplo de descrição." O bloco `Unreleased` está comentado. `SECURITY.md` marca `0.0.0` como a versão com suporte.

### Política de segurança

`SECURITY.md` pede que vulnerabilidades sejam enviadas para `contato@nataliagranato.xyz`, sem issue pública, com descrição, passos para reproduzir, versão afetada e impacto possível. O arquivo informa confirmação em até 48 horas, avaliação inicial em até 7 dias, atualizações regulares e crédito quando a correção sair.

O mesmo arquivo afirma revisão de código obrigatória, atualização via Dependabot e análise com CodeQL, e lista GitHub Advanced Security, Dependabot, CodeQL Analysis, Trivy e Checkov.

O que os arquivos e a lista de workflows confirmam:

- Dependabot está em `.github/dependabot.yml`.
- Trivy está em `.github/workflows/trivy.yml` (geração de SBOM no formato `github`).
- CodeQL aparece como workflow dinâmico `dynamic/github-code-scanning/codeql`, sem YAML no repositório. O detalhe da configuração padrão está pendente (HTTP 403 na API de default setup).
- Checkov está habilitado como ferramenta do CodeRabbit. Não há workflow Checkov na árvore atual.
- "Revisão de código obrigatória" e "GitHub Advanced Security" não são definidos por nenhum arquivo deste repositório. O que existe em arquivo é `.github/CODEOWNERS`.

### Código de conduta

`CODE_OF_CONDUCT.md` adapta o [Contributor Covenant](https://www.contributor-covenant.org/version/2/1/code_of_conduct.html) versão 2.1. Relatos vão para `contato@nataliagranato.xyz`. O arquivo descreve os níveis de ação corretiva, advertência, banimento temporário e banimento permanente.

Na seção "4. Banimento Permanente", um parágrafo de atribuição aparece antes do impacto e da consequência. Esse parágrafo usa o link `[homepage]`, que não tem definição no arquivo, e a sintaxe `(v2.1)[url]`. A seção "Atribuição", no fim, aponta a URL correta do Contributor Covenant 2.1.

### Licença

`LICENSE` é o texto integral da GNU General Public License v3, de 29 de junho de 2007. Não há, no repositório, um aviso de copyright do projeto separado desse texto. A licença informada pelo GitHub para o repositório é GPL-3.0.

## Como contribuir

O passo a passo está em [`CONTRIBUTING.md`](CONTRIBUTING.md):

1. Abrir uma issue com o máximo de detalhe possível, inclusive passos para reproduzir, quando for o caso. O template em `.github/ISSUE_TEMPLATE/BUG_REPORT.md` é o único modelo de issue versionado. Não há `config.yml` de Issue Forms.
2. Fazer fork, clonar, criar uma branch, alterar os arquivos, commitar, enviar a branch e abrir pull request contra `main`.

Dois pontos do guia não têm correspondência no código:

- A URL de clone em `CONTRIBUTING.md` usa `repository_sample`. A URL deste repositório é `https://github.com/Tech-Preta/repository-sample.git`.
- O guia pede para adicionar testes e garantir que eles cubram a mudança. Não há arquivos de teste nem job de CI que execute testes.

Ao abrir pull request, o modelo em `.github/ PULL_REQUEST_TEMPLATE/PULL_REQUEST_TEMPLATE.md` é o texto versionado para a descrição. Pelo caminho com espaço, a aplicação automática pelo GitHub continua pendente.

## Contato

- Segurança, código de conduta e o comentário automático de issues: `contato@nataliagranato.xyz`
- Proprietário em `.github/CODEOWNERS` e assignee do template de bug: [@nataliagranato](https://github.com/nataliagranato)
- Financiamento: entradas de `.github/FUNDING.yml` (`github`, `ko_fi` e `buy_me_a_coffee` com o usuário `nataliagranato`, mais a chave Pix só naquele arquivo)

Feito por [Natália Granato](https://github.com/nataliagranato).
