---
name: ctx-repo-check
description: Audita um repositório ctx-* e reporta divergências em relação à estrutura canônica do design doc de estrutura — pastas ausentes, seções obrigatórias faltando, CLAUDE.md desatualizado. Use antes de gerar artefatos ou após atualização do design doc.
globs: ["**/*"]
user-invocable: true
argument-hint: []
---

# ctx-repo-check

Audita o repo atual contra o padrão definido no design doc de estrutura do seu projeto e sinaliza o que está faltando, divergindo ou desatualizado.

## Quando ativar

- Dev pediu "checar repo", "auditar estrutura", "ctx-repo-check"
- Antes de iniciar geração de artefatos em um repo que não passou por `ctx-repo-init` recentemente
- Após o design doc de estrutura (ou a RFC que o acompanha) receber atualização — verificar se repos existentes precisam de ajuste

## Checklist de auditoria

### Pastas obrigatórias

| Pasta | Obrigatória | Observação |
|---|---|---|
| `topics/` | Sim | Cada subpasta é um tópico, na raiz do repo (não sob `docs/`) |
| `inputs/`, `working-docs/` soltos na raiz | Não deveria — sinalizar | Pré-tópico exploratório na raiz é divergência; deve estar em `.workspace/` (livre, gitignored) ou dentro de um tópico existente |
| `.workspace/` | Não | Opcional, gitignored — espaço livre do dev (drafts, ensaios, plano do code-agent, outputs de repomix). Sem definição de conteúdo obrigatório — não sinalizar o que está dentro |

### Estrutura por tópico

Para cada `topics/<t>/` encontrado, verificar:

| Item | Obrigatório | Observação |
|---|---|---|
| `README.md` | Sim | Entrada para agente — o que o tópico cobre, links internos |
| `README.md` enxuto (referência + propósito + andamento + to-dos + explicação breve de cada pasta) | Não deveria | Sinalizar quando o README carrega conteúdo denso/estrutural que pertence a `ai/`/`human/` (regras técnicas do padrão, árvore de pastas repetida, especificação longa) — reportar como aviso e sugerir extrair para `ai/`/`human/` via `ctx-repo-fix`, linkando de volta pelo campo "Entrada para agente" |
| `human/`, `ai/`, `madrs/`, `confluence/`, `.workspace/` | Não todas simultaneamente | `human/`/`ai/` não têm subpasta de tipo fixa — o dev nomeia livremente dentro delas |
| Tópico ainda sob `docs/topics/<t>/` em vez de `topics/<t>/` | Não — sinalizar | Estrutura antiga; reportar como divergência a corrigir via `ctx-repo-fix` |
| Conteúdo em `topics/<t>/analysis/`, `.../handbooks/` (sem `human`/`ai` acima), `.../runbooks/` soltas | Não — sinalizar | Estrutura antiga; reportar como divergência a corrigir via `ctx-repo-fix` |
| Subpastas de tipo fixo dentro de `human/`/`ai/` (`handbooks/`, `runbooks/`, `design-docs/`, `rfcs/`, `assessments/`) | Não — sinalizar | Essas subpastas deixaram de ser regra estrutural — o dev pode continuar usando esses nomes por convenção própria (não é erro manter), mas não exigir nem impor a quem não usa |
| Qualquer pasta vazia dentro de um tópico (sem arquivo real, ou só com `.gitkeep`), exceto `human/`, `ai/`, `madrs/`, `confluence/`, `.workspace/` na raiz do próprio tópico | Não deveria — sinalizar como bloqueante | Pasta vazia com `.gitkeep` só é legítima nas 5 pastas de raiz de tópico documentadas no README — nelas o `.gitkeep` sustenta a pasta enquanto não há conteúdo. Qualquer subpasta abaixo dessas (tipo fixo ou não) vazia é resíduo — não criar `.gitkeep` preventivo dentro de subpastas de conteúdo; se a subpasta esvaziar (todo conteúdo movido/deletado), removê-la junto via `ctx-repo-fix`, nunca deixar a casca com `.gitkeep` |
| `working-docs/` (com ou sem `drafts/`) | Não — sinalizar | Pasta separada por status foi eliminada — todo documento vive em `human/`/`ai/`, e o campo `Status` (`wip`/`in_review`/...) já diz tudo. Reportar como aviso e sugerir migração via `ctx-repo-fix` (mover conteúdo para `human/`/`ai/` pela audiência, artefatos não-documentais para `.workspace/`) |
| `madrs/` como subpasta de `human/` ou `ai/` (`human/madrs/`, `ai/madrs/`) | Não — sinalizar | `madrs/` é pasta irmã de `human`/`ai`, sem distinção de audiência. Reportar como aviso e sugerir migração via `ctx-repo-fix` |
| `.local/` em vez de `.workspace/` | Não — sinalizar | Rename de convenção antiga; reportar como aviso |
| `plans/`/`essays/` soltos fora de `.workspace/` | Não deveria — sinalizar | Devem estar em `.workspace/plans/`, `.workspace/essays/` (gitignored) |
| Decisões/ADRs em `inputs/decisions/` (raiz do repo) | Não deveria — sinalizar | Decisões fechadas pertencem a `topics/<t>/madrs/`, não à raiz do repo. Reportar como bloqueante e sugerir migração via `ctx-repo-fix` |

### MADRs (por tópico)

Para cada `topics/<t>/madrs/` populada (mais de zero arquivos além do índice):

| Item | Obrigatório | Observação |
|---|---|---|
| Nome de arquivo `MADR-NNN-<slug>.md` | Sim | Numeração sequencial dentro do tópico; qualquer outro padrão (`ADR-NNN`, slug sem número) é divergência a reportar como aviso |
| `README.md` índice na pasta `madrs/` | Sim | Tabela MADR × Status × resumo de 1 linha — sem ele não há rastreamento rápido do estado de todas as decisões |
| `**Status:**` de cada MADR usa o vocabulário oficial | Sim | `wip` \| `in_review` \| `approved` \| `not_approved` \| `deprecated` — qualquer outro valor (`DECIDIDO`, `EM ABERTO`, `SUPERSEDED`, etc.) é divergência a reportar como aviso |
| MADR com status `deprecated` tem link para a decisão sucessora | Sim | Ausência de link é bloqueante — "deprecated" sem apontar para onde a decisão vigente vive quebra a rastreabilidade |
| Links internos entre MADRs usam o nome novo (`MADR-NNN-<slug>.md`) | Sim | Referências ao slug antigo sem número (`[slug](slug.md)`) são divergência a reportar como aviso — quebram após rename |
| Decisão fechada (linguagem de "decidido"/"aprovado"/tabela "Decisões tomadas") dentro de `ai/`/`human/` (ex.: `current-status.md`, `assessments/synthesis.md`) sem MADR correspondente | Não deveria — sinalizar | Divergência de processo, não de estrutura: decisão real represada fora de `madrs/` é a mesma dívida que motiva retrofit em massa se acumular — reportar como aviso, sugerir que o dev confirme e o agente extraia o MADR na mesma sessão em que notar, sem varredura retroativa não pedida |

### Arquivos obrigatórios

| Arquivo | Obrigatório | O que verificar |
|---|---|---|
| `CLAUDE.md` | Sim | Seções: Propósito, Estrutura, **Tópicos** (com coluna "Cobre / responde"), **Regra de uso de contexto**, Convenções, Contexto de negócio |
| `AGENTS.md` | Sim | Seções: Propósito, **Regra de uso de contexto**, **Tópicos** (com coluna "Cobre / responde"), Estrutura, Restrições |
| `CHANGELOG.md` | Sim | Presente na raiz; tem pelo menos uma entrada; formato "## AAAA-MM-DD" por seção, sem categorias Added/Changed/Removed em inglês |

As seções `Regra de uso de contexto` e `Tópicos` são obrigatórias em ambos os arquivos — reportar ausência como bloqueante. A coluna **"Cobre / responde"** ausente na tabela de Tópicos é aviso (não bloqueante) — sem ela o code-agent não consegue navegar direto ao tópico certo.

### CHANGELOG.md atualizado

| Item | Obrigatório | Observação |
|---|---|---|
| Existe entrada cobrindo a versão vigente do design doc de estrutura | Sim | Se o repo já está numa versão mais nova do design doc mas o CHANGELOG.md não menciona essa versão, é divergência — sinalizar como aviso |
| Última mudança estrutural detectada via `git log` tem entrada correspondente | Não (best-effort) | Comparar commits que tocam estrutura de pastas/design doc recentes contra o CHANGELOG — sinalizar como aviso se claramente desatualizado, sem tentar reconstruir o histórico completo automaticamente |

### Consistência entre CLAUDE.md e AGENTS.md

- Ambos devem ter a mesma regra de pastas (dentro de tópico: human/ai/madrs/confluence=pastas versionadas, sem subpasta de tipo fixo, todo arquivo com Status; .workspace=livre/gitignored; raiz: inputs/working-docs soltos=divergência)
- Divergências de conteúdo entre os dois devem ser sinalizadas

### Arquivos ambíguos

- Qualquer arquivo/pasta na raiz do repo (exceto `CLAUDE.md`, `AGENTS.md`, `CHANGELOG.md`, `.gitignore`, `README.md`, `.workspace/`, `repomix.config.json`, `Makefile`, `prompts/`) é ambíguo — sinalizar. `.workspace/` não é obrigatório — sua ausência não é divergência, só sua presença desalinhada com a política (ver seção seguinte)
- Qualquer arquivo em `.claude/` que não seja `skills/` é ambíguo — sinalizar
- Qualquer conteúdo encontrado fora de `topics/<t>/` na raiz do repo (ex.: `docs/topics/<t>/`, `docs/handbook/`, `docs/ai/rfc/`, `docs/rm-odp/` de topo) é divergência de estrutura antiga — reportar como bloqueante, sugerir redistribuir para o tópico dono ou criar um tópico novo
- `private/`, `essays/`, `plans/` soltos na raiz ou dentro de um tópico (fora de `.workspace/`) é divergência — reportar como aviso e sugerir mover para `.workspace/` via `ctx-repo-fix`
- Arquivo em `human/`/`ai/` sem campo `**Status:**` no cabeçalho é divergência — reportar como aviso, nunca presumir `approved` por omissão

## Output

```
## ctx-repo-check — <nome do repo>

### Bloqueantes (impedem uso seguro como base de geração)
- [ ] <item ausente ou incorreto>

### Avisos (divergências não-bloqueantes)
- [ ] <item a corrigir>

### Ambíguo (verificar com dev)
- <arquivos fora das pastas padrão>

### OK
- [x] <item conforme>

### Resumo
Status: PRONTO | REQUER CORREÇÃO | BLOQUEADO
Bloqueantes: N | Avisos: N | Ambíguos: N
```

Se `Status: REQUER CORREÇÃO` ou `BLOQUEADO`, sugerir rodar `/ctx-repo-fix`.

## Anti-patterns

- ❌ Reportar só o que falta sem listar o que está correto
- ❌ Marcar como OK sem verificar o conteúdo das seções (não só a existência do arquivo)
- ❌ Ignorar arquivos ambíguos na raiz
</content>
