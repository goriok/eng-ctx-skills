---
name: ctx-repo-fix
description: Aplica correções em um repositório ctx-* com base no relatório do ctx-repo-check — cria pastas ausentes, preenche seções obrigatórias, sincroniza CLAUDE.md, AGENTS.md e CHANGELOG.md com a versão atual do design doc de estrutura. Use após ctx-repo-check ou quando o design doc foi atualizado.
globs: ["**/*"]
user-invocable: true
argument-hint: []
---

# ctx-repo-fix

Corrige divergências em um repo `ctx-*` identificadas pelo `ctx-repo-check` ou por atualização do design doc de estrutura (ou da RFC que o acompanha).

## Quando ativar

- Dev pediu "corrigir repo", "ctx-repo-fix", "atualizar estrutura"
- `ctx-repo-check` reportou status `REQUER CORREÇÃO` ou `BLOQUEADO`
- O design doc de estrutura (ou a RFC associada) foi atualizado e repos existentes precisam ser alinhados

## Pré-condições

1. Já está dentro do diretório do repo
2. Rodar `ctx-repo-check` primeiro se ainda não foi rodado nesta sessão — usar o relatório como input

## Workflow

### 1 — Identificar o que corrigir

Se o relatório do `ctx-repo-check` estiver disponível na conversa, usar como lista de trabalho.
Caso contrário, executar internamente o mesmo checklist do `ctx-repo-check` antes de prosseguir.

### 2 — Pastas ausentes

Para cada pasta obrigatória ausente num tópico:

```bash
mkdir -p topics/<t>/human topics/<t>/ai topics/<t>/madrs topics/<t>/confluence
touch topics/<t>/human/.gitkeep topics/<t>/ai/.gitkeep topics/<t>/confluence/.gitkeep
```

Não criar `working-docs/` nem subpastas de tipo fixo — foram eliminadas do padrão atual.

Para um tópico existente sem `README.md`: criar usando o template de `ctx-repo-topic-add`, perguntando ao dev o que o tópico cobre.

### 2b — Conteúdo encontrado fora de tópico (estrutura antiga)

Se `ctx-repo-check` sinalizou conteúdo em `docs/handbook/`, `docs/ai/{rfc,dd}`, `docs/rm-odp/`, `docs/topics/<t>/` (path pré-tópico) ou qualquer outro caminho fora de `topics/<t>/` na raiz do repo:

1. Para cada arquivo, perguntar ao dev a qual tópico ele pertence conceitualmente (nunca assumir).
2. Se nenhum tópico existente for o dono, propor criar um tópico novo via `ctx-repo-topic-add` antes de mover.
3. Mover (`git mv`) para `topics/<topico>/human/` ou `topics/<topico>/ai/` conforme audiência (sem subpasta de tipo).
4. Garantir que o arquivo movido tem `**Status:**` no cabeçalho — se não tiver, perguntar ao dev qual status atribuir (nunca presumir `approved`).
5. Atualizar referências internas (links) apontando para o caminho antigo.

Nunca deixar conteúdo "solto" fora de um tópico — isso é o núcleo da divergência de estrutura antiga, não uma exceção tolerável.

### 2c — `working-docs/` (com ou sem `drafts/<tipo>/`) e subpastas de tipo fixo dentro de `human/`/`ai/`

Se `ctx-repo-check` sinalizou `working-docs/` (solta ou em `drafts/<tipo>/`) ou subpastas de tipo fixo (`handbooks/`, `runbooks/`, `design-docs/`, `rfcs/`, `assessments/`) dentro de `human/`/`ai/` de um tópico:

1. Para cada arquivo `.md` em `working-docs/` (solta ou em `drafts/<tipo>/`): mover (`git mv`) para `topics/<t>/human/` ou `topics/<t>/ai/` conforme a audiência do conteúdo (perguntar ao dev se não for óbvio pelo texto). **Não adicionar `Status` retroativamente** — a migração é lazy: se o arquivo já não tinha `Status`, continua sem; um agente que precisar usar esse arquivo como base de geração no futuro pergunta ao dev naquele momento.
2. Para arquivos não-documentais dentro de `working-docs/` (scripts `.sh`/`.py`, planilhas, dumps): se o arquivo é referenciado por algum `.md` que também está sendo migrado (ou por outro documento já em `human/`/`ai/`/`madrs/`), mover (`git mv`) para a mesma pasta de destino do documento que o referencia — mantém versionado, junto. Se não é referenciado por nada, perguntar ao dev se confirma mover para `.workspace/<t>/` (isso **desversiona** o arquivo — `git rm --cached`, mantém no filesystem); nunca mover para `.workspace/` sem essa confirmação explícita.
3. Subpastas de tipo fixo já existentes dentro de `human/`/`ai/` (`handbooks/`, `runbooks/`, etc.) **não são erro** — o dev pode continuar usando esses nomes por convenção própria. Não forçar achatamento se o dev não pediu; só reportar que deixaram de ser regra estrutural obrigatória.
4. Remover `working-docs/` do tópico quando ficar vazia (`rmdir` em cascata).
5. Atualizar a tabela "Documentos" do `README.md` do tópico e o `CLAUDE.md`/`AGENTS.md` locais se ainda descreverem `working-docs/drafts/` como regra.

### 2c-bis — Pastas vazias (só `.gitkeep`) fora das 5 pastas de raiz de tópico

Se `ctx-repo-check` sinalizou pasta vazia (sem conteúdo real, só `.gitkeep`) em qualquer nível abaixo de `human/`, `ai/`, `madrs/`, `confluence/` — isto é diferente do item 3 acima: aqui a subpasta não tem NENHUM arquivo real, só existe pelo `.gitkeep`.

1. `git rm -r` a pasta inteira (remove o `.gitkeep` e a pasta em si) — sem perguntar ao dev, é limpeza mecânica de resíduo, nunca perda de conteúdo real.
2. Nunca recriar essa pasta com `.gitkeep` preventivamente — nem aqui nem em `ctx-repo-topic-add`. Uma subpasta de tipo (`handbooks/`, `runbooks/`, etc.) só passa a existir quando o primeiro arquivo real é colocado nela.
3. As únicas 5 pastas onde `.gitkeep` em pasta vazia é legítimo são `human/`, `ai/`, `madrs/` (raramente vazia — normalmente tem ao menos o `README.md` índice), `confluence/` e `.workspace/` (gitignored, não recebe `.gitkeep` versionado) — todas na raiz do próprio tópico, documentadas no README do repo. `.gitkeep` em qualquer subpasta abaixo dessas é sempre resíduo.

### 2d — `plans/`/`essays/` versionados indevidamente ou soltos fora de `.workspace/`

Se `ctx-repo-check` sinalizou `plans/`/`essays/` sob controle de versão ou fora de `.workspace/`: confirmar com o dev, então mover para `.workspace/<t>/plans/` ou `.workspace/plans/` (raiz) — `.workspace/` já está no `.gitignore`, então isso desversiona automaticamente (`git rm -r --cached <pasta>` antes de mover, mantém os arquivos no filesystem).

### 2e — Decisões em `inputs/decisions/` (raiz) em vez de `topics/<t>/madrs/`

Se `ctx-repo-check` sinalizou decisões na raiz do repo:

1. Confirmar com o dev qual tópico é o dono de cada decisão (podem ser vários tópicos diferentes, se o repo tiver mais de um) — nunca assumir.
2. Para cada arquivo, verificar se está tracked no git (`git ls-files --error-unmatch <arquivo>`): mover com `git mv` os tracked, `mv` normal os untracked, para `topics/<t>/madrs/`, renomeando para `MADR-NNN-<slug>.md` (numeração sequencial dentro do tópico).
3. Atualizar todas as referências internas (`grep -rl "inputs/decisions" --include="*.md" .`) — tanto links markdown relativos (recalcular `../` pela nova profundidade) quanto menções em texto puro, e todo link entre decisões que ainda usa o slug antigo sem número.
4. Normalizar o `**Status:**` de cada MADR movido para o vocabulário oficial (`wip`/`in_review`/`approved`/`not_approved`/`deprecated`) se ainda estiver no vocabulário antigo (`DECIDIDO`/`EM ABERTO`/`SUPERSEDED`/`INDEFINIDO` etc.) — perguntar ao dev o mapeamento quando não for óbvio (ex.: "EM ABERTO" pode virar `wip` ou `in_review` dependendo se o conteúdo já está fundamentado ou ainda incompleto).
5. Garantir uma linha `**Data:**` em cada MADR (logo após `**Tópico:**`) — se não houver data explícita no conteúdo, usar `git log --follow --diff-filter=A --format=%ad --date=short -- <path-antigo>` como fonte da data de criação; nunca inventar uma data sem evidência.
6. Todo MADR `deprecated` precisa linkar para a decisão sucessora — se o conteúdo ainda não linkar, adicionar antes de finalizar.
7. Criar ou atualizar `topics/<t>/madrs/README.md` — índice com tabela MADR × Status × resumo de 1 linha.
8. Remover `inputs/decisions/` se ficar vazia (`rmdir`).

### 2f — `human/madrs/`/`ai/madrs/` em vez de `madrs/` pasta irmã

Se `ctx-repo-check` sinalizou `madrs/` como subpasta de `human/`/`ai/`:

1. Criar `topics/<t>/madrs/` se ainda não existir.
2. Mover (`git mv`) cada arquivo de `human/madrs/`/`ai/madrs/` para `topics/<t>/madrs/`, renomeando para `MADR-NNN-<slug>.md` se ainda não seguir esse padrão — perguntar ao dev a numeração quando a ordem cronológica real não for recuperável via `git log --follow --diff-filter=A --format=%aI` (ex.: usar a ordem de um índice existente como base, se houver precedente em outro tópico do repo).
3. Fundir os índices (se havia `human/madrs/README.md` e `ai/madrs/README.md` separados) num único `topics/<t>/madrs/README.md`.
4. Atualizar todo link interno entre MADRs e qualquer referência externa (`grep -rl "ai/madrs\|human/madrs" --include="*.md" .`).
5. Remover as pastas antigas quando vazias.

### 2g — `private/`, `essays/` ou `repomix.config.json` soltos na raiz, ou `.local/` em vez de `.workspace/`

Se `ctx-repo-check` sinalizou algum desses fora de `.workspace/`, ou `.local/` em vez de `.workspace/`:

1. Confirmar com o dev que o conteúdo é de fato pessoal/gerado (não conteúdo mal classificado) antes de mover — nunca assumir.
2. Criar `.workspace/` se ainda não existir; mover (`mv` — esse conteúdo tipicamente nunca foi tracked, `git mv` falha sobre arquivo untracked) para dentro. Se for rename de `.local/` para `.workspace/`, usar `mv .local .workspace` preservando as subpastas.
3. Se houver `repomix.config.json`, ajustar `output.filePath` para `.workspace/<nome-do-output>` — o path é relativo ao cwd de execução, não ao config.
4. Atualizar o `.gitignore`: substituir regras específicas (`private/`, `.local/`, `contexto-consolidado.txt`, `repomix.config.json` etc.) por uma única `.workspace/`.
5. Buscar e corrigir referências textuais ao caminho antigo (`grep -rl "private/\|\.local/" --include="*.md" .` ou equivalente para o item movido).

### 2h — README.md de tópico com conteúdo denso/estrutural (não enxuto)

Se `ctx-repo-check` sinalizou um `README.md` de tópico carregando conteúdo denso/estrutural
(regras técnicas do padrão, árvore de pastas repetida, especificação longa) em vez de ficar
enxuto (referência + propósito + andamento + to-dos + explicação breve de cada pasta):

1. Extrair o conteúdo denso para um arquivo novo em `ai/` (audiência agente/técnica) ou `human/`
   (se for mais narrativo/explicativo) — perguntar ao dev qual audiência faz mais sentido se não
   for óbvio.
2. Reescrever o `README.md` só com: referência de entrada (linkando o arquivo criado no passo 1),
   propósito em 1-2 linhas, andamento, to-dos se houver, e uma tabela "O que tem em cada pasta".
3. Atualizar o campo "Entrada para agente" do README para apontar ao arquivo técnico novo, não
   mais "este arquivo".
4. Se o tópico também tinha um espelho no seu wiki (Confluence ou equivalente) do mesmo conteúdo,
   avaliar se a página precisa de correção equivalente — mas isso é uma ação separada, só sob
   pedido do dev (edição de página segue a skill de atualização de wiki que seu projeto usar, com
   autorização por página).

### 3 — CLAUDE.md ausente ou incompleto

**Se ausente:** criar do zero usando o template do `ctx-repo-init`, perguntando ao dev as seções que não são inferíveis (Propósito, Convenções, Escopo, e a tabela de Tópicos existentes).

**Se presente mas faltando seções:** adicionar apenas as seções ausentes sem reescrever o que já existe. A seção `Regra de uso de contexto` deve ser inserida com o texto canônico:

```
## Regra de uso de contexto
Dentro de cada tópico: `human/`, `ai/`, `madrs/` e `confluence/` são as pastas versionadas.
`madrs/` registra decisões arquiteturais (MADR), sem distinção de audiência — cada decisão é
um arquivo numerado `MADR-NNN-<slug>.md`, com um `README.md` índice (MADR × status × resumo).
Decisões fechadas nunca ficam em `inputs/decisions/`.
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
```

A seção `Tópicos` (tabela com colunas Tópico/Status/**Cobre-responde**/Entrada) deve ser inserida ou completada se a coluna "Cobre / responde" estiver ausente — perguntar ao dev o resumo de 1 linha por tópico se não for inferível do `README.md` de cada um.

### 3b — CHANGELOG.md ausente

**Se ausente:** criar com o template canônico (ver o design doc de estrutura do seu projeto), com uma primeira entrada reconstruída a partir do `git log` — buscar commits que tocaram estrutura de pastas ou mencionem o design doc na mensagem (`git log --oneline --all -- CLAUDE.md AGENTS.md` e uma busca equivalente por grep na mensagem de commit), sintetizando cada migração relevante em uma entrada por data. Perguntar ao dev antes de assumir uma síntese de commits ambíguos — não inventar descrição de mudança que não seja claramente estrutural.

**Se presente mas desatualizado** (`ctx-repo-check` sinalizou aviso): perguntar ao dev quais mudanças estruturais recentes faltam antes de adicionar — nunca inferir sozinho o que é "estrutural o suficiente" para entrar.

### 4 — AGENTS.md ausente ou incompleto

Mesma lógica do CLAUDE.md. Seção `Regra de uso de contexto` canônica para o AGENTS.md:

```
## Regra de uso de contexto
- Dentro de cada tópico (topics/<t>/): human/, ai/, madrs/, confluence/ são as pastas versionadas. madrs/ registra decisões arquiteturais (MADR), sem distinção de audiência — cada decisão é MADR-NNN-<slug>.md, numerado, com um README.md índice. Nunca em inputs/decisions/.
- human/ e ai/ não têm subpasta de tipo fixa — o dev nomeia livremente. Únicas regras: (1) audiência declarada pela pasta (human/ ou ai/); (2) todo arquivo tem **Status:** no cabeçalho, vocabulário fechado: wip | in_review | approved | not_approved | deprecated. Uso como base de geração: approved → direto; in_review → pode usar, sinalizando no artefato que a fonte não fechou; wip/not_approved/deprecated → checar com o dev. Nenhuma promoção automática.
- .workspace/ (raiz do repo ou de um tópico) → gitignored, o dev organiza como quiser; nada dentro dele é citável em artefatos gerados. Um artefato não-documental (script, planilha, dump) referenciado por um documento de human/ai/madrs fica versionado junto dele, na mesma pasta — só vai para .workspace/ o que não é referenciado por nenhum documento.
- Não há inputs/ nem working-docs/ — material pré-tópico ainda não classificado, ou rascunho solto, fica em .workspace/ (não versionado) ou em human/ai com status wip.
- Dúvida sobre a natureza de um asset, ou a qual tópico pertence → pergunte ao dev.
```

### 5 — Sincronizar com o design doc de estrutura atualizado

Se a motivação for atualização do design doc de estrutura, verificar:
- A regra de pastas continua a mesma? Se mudou, atualizar a seção `Regra de uso de contexto` em ambos os arquivos.
- Houve conteúdo encontrado fora de `topics/<t>/`? Redistribuir conforme passo 2b.
- A lista de skills mudou (skills antigas aposentadas, novas skills introduzidas)? Informar o dev — não instalar/remover skills automaticamente.
- Uma nova versão do design doc foi aplicada? Adicionar a entrada correspondente no `CHANGELOG.md` do repo, além de sincronizar CLAUDE.md/AGENTS.md.

### 6 — Arquivos ambíguos na raiz

Para cada arquivo ambíguo identificado, perguntar ao dev:
> "`<arquivo>` está na raiz do repo — pertence a algum tópico existente, precisa de tópico novo, deveria ir para `.workspace/`, ou pode ser removido?"

Mover conforme a decisão do dev. Nunca mover sem confirmação. Se o destino for `human/`/`ai/` de um tópico, garantir que o arquivo ganha `**Status:**` no cabeçalho antes de finalizar.

### 7 — Commit das correções

```bash
git add <arquivos alterados>
git commit -m "chore: alinha repo com o design doc de estrutura (<o que foi corrigido em uma linha>)"
```

Se a motivação for atualização do design doc, usar:
```bash
git commit -m "chore: sincroniza com design doc de estrutura v<versão> — <o que mudou>"
```

### 8 — Confirmar com ctx-repo-check

Após as correções, rodar `/ctx-repo-check` novamente. Reportar o novo status ao dev.

## Regras de ouro

- Nunca reescrever seções existentes do CLAUDE.md — apenas adicionar o que falta.
- Nunca mover arquivos sem confirmação explícita do dev.
- Nunca instalar ou atualizar skills automaticamente — só informar o dev.
- Se uma correção exigir decisão de conteúdo (ex.: qual o Propósito do repo), perguntar antes de escrever.

## Anti-patterns

- ❌ Reescrever CLAUDE.md inteiro quando só uma seção estava faltando
- ❌ Mover arquivos da raiz sem perguntar ao dev
- ❌ Commitar sem descrever o que foi corrigido
- ❌ Encerrar sem rodar `ctx-repo-check` para confirmar que o status mudou
</content>
