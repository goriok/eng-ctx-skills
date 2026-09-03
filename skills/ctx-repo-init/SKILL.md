---
name: ctx-repo-init
description: Inicializa a estrutura canônica de um repositório ctx-* — pastas, CLAUDE.md, AGENTS.md, CHANGELOG.md e commit inicial. Use em um repo recém-clonado ou vazio.
globs: ["**/*"]
user-invocable: true
argument-hint: [topico]
---

# ctx-repo-init

Configura um repositório `ctx-<topico>` seguindo o padrão definido no design doc de estrutura do seu projeto.

## Quando ativar

- Dev pediu "inicializar repo de contexto", "ctx-repo-init", "setup do repo de contexto"
- Repo recém-criado no GitLab (ou equivalente) ainda sem estrutura

## Pré-condições

1. Já está dentro do diretório do repo (`cd ctx-<topico>`)
2. Git inicializado (`git init` ou clone feito)
3. Se `$ARGUMENTS` fornecido, usa como `<topico>`; caso contrário, infere do nome do diretório atual

## Workflow

### 1 — Confirmar tópico

Extrair `<topico>` de `$ARGUMENTS` ou do nome do diretório atual. Confirmar com o dev antes de prosseguir.

### 2 — Criar estrutura de pastas

```bash
mkdir -p topics .claude/skills
touch topics/.gitkeep
printf '.workspace/\n.repomix/\n' >> .gitignore
```

Todo conteúdo vive dentro de um tópico em `topics/<topico>/`, na raiz do repo — sem `docs/` intermediário nem qualquer path de topo além de `topics/`. Não há `inputs/` nem `working-docs/` na raiz do repo — material pré-tópico ainda não classificado fica em `.workspace/` (gitignored, livre, nunca versionado). `.repomix/` (gitignored, separado de `.workspace/`) recebe só o output do repomix — ver passo 4c. Um repo recém-inicializado começa com `topics/` vazio; o primeiro tópico é criado via `ctx-repo-topic-add` quando houver conteúdo real para organizar.

Criar subpasta opcional somente se o dev confirmar que o escopo do repo inclui:
- `.claude/skills/` — skills locais

### 3 — Escrever CLAUDE.md

Preencher o template abaixo com as respostas do dev. Perguntar apenas o que não for inferível:

```markdown
# CLAUDE.md — ctx-<topico>

## Propósito
<Uma linha: o que este repo de contexto contém e para quê.>

## Estrutura
\`\`\`
ctx-<topico>/
└── topics/<t>/
    ├── README.md    # entrada para agente
    ├── human/       # linguagem humana, todo arquivo com Status
    ├── ai/          # linguagem densa p/ agentes, todo arquivo com Status
    ├── madrs/       # decisões arquiteturais, MADR-NNN-<slug>.md + README.md índice
    └── confluence/  # espelhos curados (ou o equivalente no seu wiki)
\`\`\`

`.workspace/` (raiz do repo ou de um tópico) é gitignored, livre, nunca versionado — não entra
na árvore acima porque não é conteúdo do padrão, é espaço de trabalho do dev.

`README.md` de tópico é enxuto: referência de entrada + propósito + andamento + to-dos +
explicação breve de cada pasta. Conteúdo denso/estrutural vai para `ai/` ou `human/`.

## Tópicos (topics/)
| Tópico | Status | Cobre / responde | Entrada |
|---|---|---|---|
| _(nenhum ainda — use ctx-repo-topic-add)_ | | | |

## Regra de uso de contexto
Dentro de cada tópico: `human/`, `ai/`, `madrs/` e `confluence/` são as pastas versionadas.
`madrs/` registra decisões arquiteturais (MADR), sem distinção de audiência — cada decisão é um
arquivo numerado `MADR-NNN-<slug>.md`, com um `README.md` índice. Decisões fechadas nunca ficam
em `inputs/decisions/` na raiz.
`human/` e `ai/` não têm subpasta de tipo fixa — o dev nomeia livremente. Únicas regras: (1)
audiência declarada pela pasta (`human/` ou `ai/`); (2) todo arquivo tem `**Status:**` no
cabeçalho, vocabulário fechado: `wip` | `in_review` | `approved` | `not_approved` | `deprecated`.
Uso como base de geração: `approved` → direto; `in_review` → pode usar, sinalizando no artefato
que a fonte não fechou; `wip`/`not_approved`/`deprecated` → checar com o dev. Nenhuma promoção
automática — o status só muda quando o dev edita o arquivo.
`.workspace/` (raiz do repo ou de um tópico) é gitignored — o dev organiza como quiser; nada
dentro dele é citável em artefatos gerados. Um artefato não-documental (script, planilha, dump)
referenciado por um documento de `human/`/`ai/`/`madrs/` fica versionado junto dele, na mesma
pasta — só vai para `.workspace/` o que não é referenciado por nenhum documento.
Não há `inputs/` nem `working-docs/` — material pré-tópico ainda não classificado, ou rascunho
solto, fica em `.workspace/` (não versionado) ou em `human/`/`ai/` com status `wip`.
Se a natureza de um asset for ambígua, ou não pertencer a nenhum tópico existente, perguntar ao dev antes de usar ou criar um tópico novo.

## Convenções
<Convenções específicas: formato de arquivo, padrão de nomes, fonte do seu wiki se aplicável.>

## Contexto de negócio
- **Time:** {SEU_TIME} — use seu time como referência aqui
- **Escopo deste repo:** <descrever>
- **Restrições:** <compliance/regulação relevante ao seu contexto, se houver>
```

### 4 — Escrever AGENTS.md

```markdown
# AGENTS.md — ctx-<topico>

## Propósito
<Uma linha.>

## Regra de uso de contexto
- Dentro de cada tópico (topics/<t>/): human/, ai/, madrs/, confluence/ são as pastas versionadas. madrs/ registra decisões arquiteturais (MADR), sem distinção de audiência — cada decisão é MADR-NNN-<slug>.md, numerado, com um README.md índice. Nunca em inputs/decisions/.
- human/ e ai/ não têm subpasta de tipo fixa — o dev nomeia livremente. Únicas regras: (1) audiência declarada pela pasta (human/ ou ai/); (2) todo arquivo tem **Status:** no cabeçalho, vocabulário fechado: wip | in_review | approved | not_approved | deprecated. Uso como base de geração: approved → direto; in_review → pode usar, sinalizando no artefato que a fonte não fechou; wip/not_approved/deprecated → checar com o dev. Nenhuma promoção automática.
- .workspace/ (raiz do repo ou de um tópico) → gitignored, o dev organiza como quiser; nada dentro dele é citável em artefatos gerados. Um artefato não-documental (script, planilha, dump) referenciado por um documento de human/ai/madrs fica versionado junto dele, na mesma pasta — só vai para .workspace/ o que não é referenciado por nenhum documento.
- Não há inputs/ nem working-docs/ — material pré-tópico ainda não classificado, ou rascunho solto, fica em .workspace/ (não versionado) ou em human/ai com status wip.
- Dúvida sobre a natureza de um asset, ou a qual tópico pertence → pergunte ao dev.
- README.md de tópico é enxuto: referência de entrada + propósito + andamento + to-dos + explicação breve de cada pasta. Conteúdo denso/estrutural vai para ai/ ou human/.

## Tópicos (topics/)
| Tópico | Status | Cobre / responde | Entrada |
|---|---|---|---|
| _(nenhum ainda — use ctx-repo-topic-add)_ | | | |

## Estrutura
- topics/<t>/: cada tópico com sua árvore human/ai/madrs/confluence
- .workspace/: livre, gitignored, opcional (raiz ou dentro de um tópico)

## Restrições
- Não expor conteúdo `wip`/`not_approved`/`deprecated` de topics/<t>/human/ ou ai/ fora do repo sem checar com o dev.
- Se houver compliance/regulação aplicável ao seu contexto, não gerar artefatos de decisão a partir de conteúdo `wip` sem validação do dev.
```

### 4b — Escrever CHANGELOG.md

Todo repo `ctx-*` mantém um `CHANGELOG.md` na raiz — registra apenas mudanças ESTRUTURAIS (migrações de estrutura, bumps de versão do design doc de estrutura, tópicos criados/removidos, decisões em MADR); não lista documentos individuais criados em `human/`/`ai/` nem mudanças de `Status`, isso já está no `git log`. Ver o design doc de estrutura do seu projeto (se existir) para o template completo.

```markdown
# Changelog

Mudanças estruturais deste repositório — migrações de estrutura, versões do design doc de
estrutura aplicadas, tópicos criados/removidos, decisões registradas em MADR. Não lista
documentos individuais criados em `human/`/`ai/` nem mudanças de `Status` — ver `git log` para
isso.

## <data de hoje>

- Setup inicial do repositório conforme design doc de estrutura v<versão vigente>.
```

### 4c — Repomix (opcional — perguntar ao dev se o repo já quer isso desde o início)

Se o dev confirmar, criar `repomix.config.json` na raiz, `Makefile` com os alvos
`repomix`/`repomix-topics`/`clean-repomix`, e `scripts/repomix-per-topic.sh` — usar como
referência qualquer repo `ctx-*` já existente com esses três arquivos, copiando verbatim (eles
não variam entre repos, exceto o `headerText` do config, que deve refletir o domínio do novo
repo). Se o repo tiver um tópico dedicado a explicar o próprio padrão de engenharia de contexto,
ver lá o que cada arquivo faz e por quê.

Se o dev preferir adicionar depois, pular este passo — não é obrigatório ter repomix desde a
inicialização.

### 5 — Adicionar tópico/tag de descoberta (lembrete, se aplicável)

Se o seu GitLab (ou equivalente) usa tags de descoberta de repositório, informar o dev:
> Adicionar a tag de descoberta apropriada nas configurações do repositório (ex.: Settings → General → Topics), seguindo a convenção do seu grupo.

### 6 — Commit inicial

```bash
git add CLAUDE.md AGENTS.md CHANGELOG.md .gitignore topics/.gitkeep
# se o passo 4c foi feito:
git add repomix.config.json Makefile scripts/repomix-per-topic.sh
git commit -m "chore: setup inicial ctx-<topico>

Estrutura de diretórios, CLAUDE.md, AGENTS.md e CHANGELOG.md conforme design doc de estrutura."
```

## Anti-patterns

- ❌ Criar o repo sem confirmar o tópico com o dev
- ❌ Deixar `<descrever>` ou `<uma linha>` sem preencher no CLAUDE.md
- ❌ Pular o commit inicial
- ❌ Criar o repo sem `CHANGELOG.md` — mesmo com uma única entrada de setup inicial
- ❌ Criar `docs/topics/` ou qualquer path com `docs/` na frente — todo conteúdo de definição vive dentro de um tópico em `topics/<topico>/` na raiz do repo
- ❌ Adicionar subpastas não previstas no design doc de estrutura sem justificativa
</content>
