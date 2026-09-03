---
name: ctx-repo-topic-add
description: Adiciona um tópico novo a um repo de contexto ctx-* existente — cria topics/_templates/ (se ausente), instancia a árvore do tópico e registra a entrada (com "Cobre/responde") no CLAUDE.md. Use quando o dev quer materializar um tópico planejado ou criar um tópico inédito dentro do ctx atual.
globs: ["**/*"]
user-invocable: true
argument-hint: [topico]
---

# ctx-repo-topic-add

Materializa um tópico novo dentro de um repo `ctx-*` existente, seguindo o padrão do design doc
de estrutura vigente: dentro do tópico, a árvore é organizada só por **audiência** (`human/` vs
`ai/`) — sem subpasta de tipo fixa — mais `madrs/` (decisões arquiteturais, sem distinção de
audiência, pasta irmã de `human`/`ai`), `confluence/` (espelhos do seu wiki) e `.workspace/`
(livre, gitignored).

Não existe mais conteúdo fora de um tópico — todo material pertence a algum `topics/<topico>/`,
na raiz do repo. Dentro de `human/`/`ai/`, todo arquivo tem um campo `**Status:**` obrigatório
(vocabulário: `wip` | `in_review` | `approved` | `not_approved` | `deprecated`) — não há
categoria de pasta separada para isso, só o `Status`.

## Quando ativar

- Dev pediu "adiciona tópico", "cria tópico", "novo tópico", "ctx-repo-topic-add"
- Dev quer materializar um tópico que está como `Planejado` no CLAUDE.md
- Dev quer criar um tópico inédito no ctx atual
- Um asset foi encontrado sem tópico dono (ex.: durante `ctx-repo-fix`) e precisa de um novo tópico

## Pré-condições

1. Já está dentro do repo de contexto (`topics/` ou `CLAUDE.md` existe na raiz)
2. Se `$ARGUMENTS` fornecido, usa como `<topico>`; caso contrário, perguntar ao dev
3. O repo tem um CLAUDE.md com seção "Tópicos" (ou equivalente que liste o domínio)

## Workflow

### 1 — Identificar tópico e domínio

Extrair `<topico>` de `$ARGUMENTS` ou perguntar ao dev.

Ler o CLAUDE.md do repo para identificar:
- o domínio declarado (ex: "features de autenticação")
- os tópicos já listados (tabela "Tópicos", incluindo a coluna "Cobre / responde")

Se o tópico pedido parecer claramente fora do domínio do repo, avisar o dev com raciocínio explícito e aguardar confirmação antes de continuar. Exemplo de aviso:

> O repo `ctx-auth-feats` cobre autenticação. O tópico `billing` parece fora desse domínio. Confirmar que está no repo certo?

Se o tópico pedido parecer sobreposto a um tópico já existente (comparar contra a coluna "Cobre / responde" de cada linha), avisar antes de criar duplicata — pode ser melhor estender o tópico existente.

Se o tópico já existir como diretório em `topics/<topico>/`, informar o dev e interromper — não sobrescrever.

### 2 — Criar ou confirmar topics/_templates/

Verificar se `topics/_templates/` existe e segue a árvore atual (`human/`, `ai/` sem subpasta de
tipo; `madrs/`; `confluence/`; sem `working-docs/`). Se estiver desatualizado (subpastas de tipo
fixo, ou `working-docs/drafts/` presente), avisar o dev e oferecer migrar antes de prosseguir.

Se `topics/_templates/` não existir, criá-lo com o esqueleto canônico:

```bash
mkdir -p topics/_templates/human topics/_templates/ai \
         topics/_templates/madrs topics/_templates/confluence
touch topics/_templates/human/.gitkeep topics/_templates/ai/.gitkeep \
      topics/_templates/confluence/.gitkeep
```

`.workspace/` não entra no template — é criado sob demanda, livre, e nunca é commitado (está no `.gitignore` da raiz do repo).

Criar os arquivos-esqueleto embutidos abaixo (ver passo 3).

Se `topics/_templates/` já existir e estiver atualizado, usá-lo como está.

### 3 — Arquivos-esqueleto do template

Criar (ou verificar que existem) os seguintes arquivos em `topics/_templates/`:

**`CLAUDE.md`** (regra de contexto local do tópico, carregada automaticamente pelo Claude Code ao trabalhar dentro da pasta)
```markdown
# CLAUDE.md — <tópico>

## Regra de uso de contexto
`human/`, `ai/`, `madrs/` e `confluence/` são as pastas versionadas deste tópico.
`human/` e `ai/` não têm subpasta de tipo fixa — o dev nomeia livremente. Únicas regras: (1)
audiência declarada pela pasta (`human/` ou `ai/`); (2) todo arquivo tem `**Status:**` no
cabeçalho, vocabulário fechado: `wip` | `in_review` | `approved` | `not_approved` | `deprecated`.
Não há distinção estrutural de "exploração" vs. "definição" — só o `Status`.
Uso como base de geração: `approved` → direto; `in_review` → pode usar, sinalizando no artefato
que a fonte não fechou; `wip`/`not_approved`/`deprecated` → checar com o dev. Nenhuma promoção
automática — o status só muda quando o dev edita o arquivo.
`madrs/` registra decisões arquiteturais (MADR), sem distinção de audiência — cada decisão é um
arquivo numerado `MADR-NNN-<slug>.md`, com um `README.md` índice (MADR × status × resumo).
**Registrar MADR no momento da decisão, não depois.** Quando uma decisão arquitetural real for
tomada durante a sessão, criar o `MADR-NNN-<slug>.md` e atualizar o índice antes de encerrar a
tarefa, mesmo sem pedido explícito. Não represar em notas soltas para "formalizar depois".
`.workspace/` é gitignored — o dev organiza como quiser; nada dentro dele é citável em artefatos
gerados. Um artefato não-documental (script, planilha, dump) referenciado por um documento de
`human/`/`ai/`/`madrs/` fica versionado junto dele, na mesma pasta — só vai para `.workspace/`
o que não é referenciado por nenhum documento.

## Restrições
<Restrições específicas deste tópico, se houver — em branco por padrão.>
```

**`AGENTS.md`** (mesma regra, formato agnóstico — para agentes que não leem CLAUDE.md)
```markdown
# AGENTS.md — <tópico>

## Regra de uso de contexto
- human/, ai/, madrs/, confluence/ são as pastas versionadas deste tópico.
- human/ e ai/ não têm subpasta de tipo fixa — o dev nomeia livremente. Únicas regras: (1)
  audiência declarada pela pasta (human/ ou ai/); (2) todo arquivo tem **Status:** no cabeçalho,
  vocabulário fechado: wip | in_review | approved | not_approved | deprecated. Não há distinção
  estrutural de "exploração" vs. "definição" — só o Status. Uso como base de geração: approved →
  direto; in_review → pode usar, sinalizando no artefato que a fonte não fechou;
  wip/not_approved/deprecated → checar com o dev. Nenhuma promoção automática.
- madrs/ registra decisões arquiteturais (MADR), sem distinção de audiência.
- Registrar MADR no momento da decisão, não depois: toda decisão arquitetural real fechada
  durante a sessão vira MADR (+ índice atualizado) antes de encerrar a tarefa, sem esperar
  pedido explícito. Não represar em notas soltas.
- .workspace/ → gitignored, o dev organiza como quiser; nada dentro dele é citável em artefatos
  gerados. Um artefato não-documental (script, planilha, dump) referenciado por um documento de
  human/ai/madrs fica versionado junto dele, na mesma pasta — só vai para .workspace/ o que não
  é referenciado por nenhum documento.

## Restrições
<Restrições específicas deste tópico, se houver.>
```

**`README.md`** (entrada humana do tópico — enxuto: referência + propósito + andamento + to-dos
+ explicação breve de cada pasta; conteúdo denso/estrutural vai para `ai/`/`human/`, não aqui;
não duplica a regra de contexto do CLAUDE.md/AGENTS.md do tópico)
```markdown
# <tópico>

**Status:** Planejado
**Entrada para agente:** <link para o doc de ai/ que for a entrada técnica, quando existir — ou "este arquivo" enquanto não houver>

---

## Propósito

<Uma ou duas linhas: o que este tópico cobre no contexto do domínio, e o que fica de fora.>

## Andamento

<Estado atual em 1-2 linhas: o que já foi decidido, o que está em análise.>

## To-dos

- [ ] <pendência concreta, se houver — remover a seção se não houver nenhuma ainda>

## O que tem em cada pasta

| Pasta | Conteúdo |
|---|---|
| `human/` | <o que tem hoje, ou "vazia por ora"> |
| `ai/` | <o que tem hoje, ou "vazia por ora"> |
| `madrs/` | <o que tem hoje, ou "vazia por ora, só o índice"> |
| `confluence/` | <o que tem hoje, ou "vazia por ora"> |
| `.workspace/` | Gitignored, livre — não versionado |
```

**`.gitkeep`** em `human/`, `ai/` e `confluence/` enquanto vazias — sem arquivo de conteúdo além do `README.md`.

**Vocabulário de status** (usar em todo `**Status:**` de qualquer arquivo em `human/`/`ai/`/`madrs/`):

| Status | Significado |
|---|---|
| `wip` | Em elaboração — conteúdo ainda incompleto, sem decisão formada |
| `in_review` | Conteúdo completo e fundamentado, aguardando decisão formal ou confirmação externa |
| `approved` | Decisão/definição tomada e em vigor — não reabrir sem nova evidência |
| `not_approved` | Avaliada e rejeitada |
| `deprecated` | Substituída por outra — **deve linkar para a decisão/documento sucessor** no próprio Status ou logo abaixo dele; nunca deixar "deprecated" sem apontar para onde a versão atual vive |

Criar `madrs/README.md` (índice) desde já, mesmo vazio de decisões: tabela MADR × Status × resumo de 1 linha.

### 4 — Instanciar o tópico

```bash
cp -r topics/_templates/ topics/<topico>
```

Substituir `<tópico>` nos cabeçalhos dos arquivos copiados pelo nome real do tópico (README.md, CLAUDE.md e AGENTS.md do tópico).

### 5 — Registrar no CLAUDE.md/AGENTS.md da RAIZ do repo

Não confundir com o `CLAUDE.md`/`AGENTS.md` DO tópico (criados no passo 4, dentro de `topics/<topico>/`) — este passo edita os arquivos na raiz do repo, que mantêm o índice de todos os tópicos.

Localizar a tabela "Tópicos" no CLAUDE.md e AGENTS.md da raiz.

Se o tópico já estiver listado como `Planejado`, atualizar a linha:
- coluna status: breve descrição do estado inicial (ex: `Em inicialização`)
- coluna "Cobre / responde": preencher com o resumo dado pelo dev no passo 1
- coluna entrada: `topics/<topico>/README.md`

Se o tópico não estiver listado, inserir linha nova:

```
| **<topico>** | Em inicialização | <resumo de 1 linha: palavras-chave e perguntas que resolve> | [topics/<topico>/README.md](topics/<topico>/README.md) |
```

Nunca deixar a coluna "Cobre / responde" vazia — é o que torna o tópico descobrível pelo code-agent. Perguntar ao dev se não for óbvio.

### 6 — Commit

```bash
git add topics/_templates/ topics/<topico>/ CLAUDE.md AGENTS.md
git commit -m "docs: add topic scaffold for <topico>"
```

### 7 — Reportar

Informar ao dev:
- tópico criado em `topics/<topico>/`
- se `topics/_templates/` foi criado nesta execução ou já existia
- linha do CLAUDE.md/AGENTS.md atualizada/inserida, incluindo o resumo "Cobre / responde"
- próximos passos sugeridos: preencher `README.md` e começar a popular `human/`/`ai/` conforme o conteúdo for surgindo — sempre com `Status: wip` até o dev decidir fechar

## Anti-patterns

- ❌ Criar o tópico sem confirmar o nome com o dev quando `$ARGUMENTS` está ausente
- ❌ Sobrescrever `topics/<topico>/` se já existir — interromper e avisar
- ❌ Pular a validação de domínio ou de sobreposição com tópico existente
- ❌ Deixar `<tópico>` ou `<descrever>` sem substituir nos arquivos copiados
- ❌ Deixar a coluna "Cobre / responde" vazia no índice — quebra a navegação do code-agent
- ❌ Omitir o commit — a estrutura do tópico deve ser rastreada desde o início
- ❌ Criar `topics/_templates/` com subpastas de tipo fixo ou `working-docs/drafts/` — esse padrão foi eliminado do design doc de estrutura atual
- ❌ Criar o tópico sob `docs/topics/` — os tópicos vivem em `topics/` na raiz do repo
</content>
