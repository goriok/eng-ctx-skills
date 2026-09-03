---
name: ctx-repo-promptl
description: Cria, atualiza ou corrige um prompt para agent-tools externas (Gemini Gem, NotebookLM, GPT, etc.) a partir do contexto do repositório atual. Use quando o dev pedir para gerar ou atualizar um prompt para uma ferramenta externa.
globs: ["**/*"]
user-invocable: true
argument-hint: [tool-name] [create|update|fix]
---

# ctx-repo-promptl

Gera um prompt para uma agent-tool externa (Gemini Gem, NotebookLM, GPT Custom Instructions, etc.) usando o contexto real do repositório atual como base. O prompt gerado é opinado — reflete o domínio, as restrições e as convenções deste repo, não um template vazio.

## Quando ativar

- Dev pediu "gera prompt para o Gem", "atualiza o prompt do NotebookLM", "cria prompt para GPT"
- Dev pediu "ctx-repo-promptl"
- Dev quer criar, atualizar ou corrigir um prompt de ferramenta externa usando o contexto do repo

## Pré-condições

1. Estar dentro de um repositório de contexto com `CLAUDE.md` ou `AGENTS.md` na raiz
2. `$ARGUMENTS` deve conter o nome da tool alvo e a operação (`create`, `update` ou `fix`)
   - Se omitido, perguntar ao dev antes de prosseguir
3. Para `update` ou `fix`: o arquivo de prompt alvo já deve existir em `topics/<topico>/ai/` (ver passo 5 sobre qual tópico)

## Workflow

### 1 — Identificar tool e operação

Extrair de `$ARGUMENTS`:
- `tool`: nome da ferramenta (ex: `gemini-gem`, `notebooklm`, `gpt`)
- `op`: `create`, `update` ou `fix`

Se ausentes, perguntar ao dev.

### 2 — Ler o contexto do repositório

Ler **nesta ordem**, parando quando tiver contexto suficiente:

1. `CLAUDE.md` ou `AGENTS.md` — propósito, tabela de Tópicos (com "Cobre / responde"), regras de uso de contexto
2. Se o prompt é sobre o repo inteiro: percorrer o `README.md` de cada tópico em `topics/*/`
3. Se o prompt é sobre um tópico específico: ler `topics/<topico>/README.md` e os documentos em `human/`/`ai/`/`confluence/` daquele tópico

Não inventar contexto. Usar apenas o que está nos arquivos lidos.

### 3 — Identificar o perfil da tool

| Tool | Tipo de prompt | Campo alvo |
|---|---|---|
| `gemini-gem` | System prompt interativo | "System Instructions" do Gem |
| `notebooklm` | Guia de leitura (fonte injetada) | Primeira fonte carregada no notebook |
| `gpt` | Custom Instructions | "What would you like ChatGPT to know?" |
| Outra | Perguntar ao dev qual campo recebe o prompt | — |

Adaptar tom e estrutura ao perfil:
- **System prompt interativo** (Gem, GPT): imperativo, regras de comportamento, o que fazer/não fazer
- **Guia de leitura** (NotebookLM): descritivo, orienta interpretação dos documentos carregados, sugere perguntas

### 4 — Redigir o prompt

Regras obrigatórias:

- **Máximo 4000 caracteres** — contar antes de entregar; cortar se necessário, priorizando regras de comportamento sobre exemplos
- Sempre incluir a regra de **Status** (vocabulário `wip`/`in_review`/`approved`/`not_approved`/`deprecated` e como cada um pode ser usado como base de geração) — é a regra central de confiabilidade do conteúdo, não uma distinção estrutural de pastas
- Referenciar o domínio real do repo (lido no passo 2) — não usar placeholders genéricos
- Incluir restrições reais do repo (compliance, convenções de nomeação, fontes externas se houver)
- Para `update`: preservar seções que ainda fazem sentido; reescrever apenas o que mudou ou está desatualizado
- Para `fix`: identificar explicitamente o que está errado antes de corrigir

Estrutura recomendada (adaptar por tipo de prompt):

```
# [Título que identifica a tool e o domínio]

## Papel / O que é este notebook
[Uma linha sobre o que o agente/notebook cobre]

## Regra central: campo Status
[wip | in_review | approved | not_approved | deprecated — e como cada um pode ser usado]

## Contexto do repositório
[Domínio, restrições, convenções — extraídos do CLAUDE.md/topics/]

## Documentos de referência
[O que existe em cada tópico e o que cada um responde]

## Como responder / Como interpretar
[Comportamento esperado, o que citar, o que não inferir]

## O que não fazer
[Anti-patterns específicos do domínio]
```

### 5 — Salvar o arquivo

Não há `docs/` de topo nem subpasta de tipo fixo (`prompts/`) — o prompt gerado vive direto em
`ai/` do tópico a que se refere, nomeado pelo conteúdo.

- Se o prompt cobre o repo inteiro (contexto de múltiplos tópicos): perguntar ao dev qual tópico deve hospedá-lo (tipicamente o mais fundacional/amplo do repo — ex. um tópico dedicado a explicar o próprio padrão de engenharia de contexto, se existir), ou se cabe criar um tópico dedicado via `ctx-repo-topic-add`.
- Se o prompt cobre um tópico específico: caminho é `topics/<topico>/ai/<tool-name>-prompt.md`.
- Se o arquivo já existir (`update`/`fix`): sobrescrever
- Se não existir (`create`): criar, incluindo o diretório se necessário
- Adicionar o cabeçalho `context-source` (frontmatter + callout apontando para a URL GitLab do próprio arquivo) e o campo `**Status:**` no cabeçalho do arquivo, como qualquer documento de `ai/`

### 6 — Reportar

Informar ao dev:
- Caminho do arquivo gerado
- Tamanho em caracteres
- Se houve corte de conteúdo por limite de 4000 chars e o que foi removido

## Anti-patterns

- Gerar prompt com placeholders (`<descrever>`, `<seu domínio>`) sem preencher com o contexto real
- Exceder 4000 caracteres sem avisar o dev
- Inventar restrições ou convenções que não estão nos arquivos do repo
- Para `update`/`fix`: reescrever seções que ainda estão corretas
- Omitir a regra de `Status` — ela é obrigatória em todos os prompts
</content>
