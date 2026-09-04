# Como um repositório de contexto é organizado

Este documento explica a estrutura de um repositório de contexto — como as pastas e documentos
são organizados, e por que são assim. Os nomes de tópico e trechos usados como ilustração
(Cluster API, CNI, admission control, cluster-resource-set etc.) vêm de um repositório real que
segue essa convenção, não são uma lista fixa que todo repositório precisa ter.

Um repositório de contexto acumula conhecimento de domínio com apoio de ferramentas de IA
generativa. A estrutura descrita aqui existe para que esse conteúdo tenha um nível de
confiabilidade explícito, distinguindo o que é conhecimento consolidado do que ainda está em
elaboração.

## O que tem dentro de um tópico

Cada assunto técnico do domínio vive em uma pasta própria dentro de `topics/`, sempre com a mesma
estrutura interna:

```text
topics/<topico>/
├── README.md    # entrada humana: propósito, andamento, to-dos, o que tem em cada pasta
├── CLAUDE.md    # regra de uso de contexto local ao tópico
├── AGENTS.md    # mesma regra, formato agnóstico de ferramenta
├── human/       # linguagem humana, todo arquivo com Status
├── ai/          # linguagem densa para agentes, todo arquivo com Status
├── madrs/       # decisões arquiteturais, um arquivo por decisão + índice
└── .workspace/  # opcional, gitignored, nunca versionado
```

No exemplo usado neste documento, o tópico `cluster-api` tem um arquivo em `human/` explicando o
padrão operator em linguagem acessível, e dois arquivos em `ai/` com a hierarquia completa de CRDs
e evidência de código. Essa divisão em dois arquivos `ai/` é só uma escolha de organização daquele
tópico — `human/` e `ai/` se dividem apenas por audiência, sem categoria fixa de tipo de documento
dentro de cada uma.

Todo conteúdo em `human/` é conciso, propositivo, com tom explicativo — para quem está entrando
no assunto, não para quem já opera o processo. Isso inclui não pressupor que o leitor conhece um
estado anterior do domínio (afirmar o que é, não o que deixou de ser), e preferir listas a
parágrafos que espremem vários itens numa linha só.

## A única marca de confiabilidade: o campo Status

Todo arquivo em `human/` e `ai/` carrega um campo `**Status:**` no cabeçalho, com vocabulário
fechado:

- `wip` — em elaboração
- `in_review` — pronto, aguardando confirmação
- `approved` — decisão ou conteúdo em vigor
- `not_approved` — avaliado e rejeitado
- `deprecated` — substituído

Ao gerar algo a partir desse conteúdo, o uso muda conforme o Status:

- `approved` → usar direto
- `in_review` → pode usar, sinalizando que a fonte ainda não fechou
- `wip`/`not_approved`/`deprecated` → checar com o dev antes de usar

O status só muda quando alguém edita o arquivo de verdade — nenhum agente promove status sozinho,
mesmo que o conteúdo pareça correto.

## Decisões arquiteturais viram MADR no momento em que são fechadas

`madrs/` é pasta irmã de `human/`/`ai/`, sem distinção de audiência, e registra decisões
arquiteturais reais — não achados de investigação, não notas de leitura de código, só decisões de
fato tomadas. Cada uma é um arquivo numerado (`MADR-NNN-<slug>.md`), com um `README.md` índice
(MADR × status × resumo) na mesma pasta.

Toda decisão arquitetural real, quando fechada, vira MADR no mesmo momento em que é tomada — não
fica represada em notas soltas para "formalizar depois". No exemplo deste documento, se o time
decidisse formalmente adotar `ClusterAddonProvider`/CAAPH no lugar do `ClusterResourceSet` atual,
essa decisão — não a pesquisa que a embasou — viraria um MADR.

## As pastas nunca ficam vazias como esqueleto morto

As quatro pastas de raiz de um tópico (`human/`, `ai/`, `madrs/`, `.workspace/`) podem existir
vazias por um tempo, esperando o primeiro conteúdo real. Mas qualquer subpasta abaixo delas — por
exemplo, um futuro `ai/runbooks/` com passo a passo operacional — só nasce quando o primeiro
arquivo de verdade é colocado nela, nunca antes, como esqueleto vazio.

## `.workspace/`: onde o rascunho vive antes de virar documento

`.workspace/` é gitignored — completamente livre, organizado do jeito que o dev quiser, e nunca
citável como fonte em nenhum documento gerado. É o lugar de anotações soltas, rascunhos de
investigação, ou material bruto (como uma transcrição de reunião) antes de virar um documento real
em `human/`/`ai/`. O que não é referenciado por nenhum documento publicado fica lá; o que vira
evidência citável precisa estar dentro de um documento versionado de verdade, com Status.

## Ferramental e skills que operam essa estrutura

As skills `ctx-repo-*` deste repositório ([README.md](../README.md)) automatizam a operação dessa
estrutura — inicializar, auditar, corrigir, adicionar tópico — sem substituir a decisão de
adotá-la. Ver [ferramental.md](ferramental.md) para o ferramental de apoio (busca semântica,
rastreamento de consumo de token) que opera em conjunto com as skills.
