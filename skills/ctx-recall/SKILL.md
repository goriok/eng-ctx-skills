---
name: ctx-recall
description: Busca semântica local via recall (Qdrant + Ollama) sobre qualquer projeto/documentação já indexada — incluindo repositórios de contexto ctx-*. Use antes de grep/leitura manual quando a pergunta é conceitual e a localização exata do conteúdo não é óbvia.
globs: ["**/*"]
---

# Busca semântica local (recall)

`recall` indexa Markdown de projetos configurados em `~/.config/recall/recall.toml` num Qdrant
local (embeddings via Ollama `nomic-embed-text`), uma collection por projeto. Prefira isto a
grep quando a pergunta é conceitual ("como funciona X", "onde decidimos Y") e não se sabe de
antemão em qual arquivo a resposta está — não é específico de nenhum repositório em particular,
qualquer projeto pode estar indexado.

Ferramenta: [goriok/recall](https://github.com/goriok/recall).

## Comando

```bash
recall search "<query em linguagem natural>" --in <collection> --top 5
```

- `--in <collection>` restringe a uma collection — omitir para buscar em todas as configuradas
  (ranking por score global, sem normalização entre collections).
- `--top N` controla quantos chunks retornam (default 5).
- Query em português ou inglês, frase livre — não precisa ser palavra-chave exata.

## Descobrir o que está indexado

Não presumir nomes de collection — verificar antes de usar `--in`:

```bash
recall collections list
```

Lista todas as collections do Qdrant com contagem de vetores. Rodar uma busca sem `--in`
primeiro também é uma forma válida de descobrir onde o conteúdo relevante está, já que o
resultado traz a collection de origem de cada chunk.

`--in` com nome de collection errado ou inexistente não gera erro — retorna silenciosamente
`No results found.` (o searcher ignora exceções por collection ausente). Se uma busca que
deveria achar algo vier vazia, conferir o nome exato antes de concluir que o conteúdo não existe.

## Como interpretar o resultado

Cada resultado retorna o `source` (path absoluto do arquivo original) e o texto do chunk
(heading + corpo). Sempre citar/abrir o arquivo fonte antes de usar o conteúdo como base de
geração — o chunk não carrega metadado do documento (como um campo de status, se o projeto usar
um), então confirmar diretamente no arquivo qualquer convenção de confiabilidade que o projeto
indexado use antes de tratar o conteúdo como definitivo.

## Quando NÃO usar

- Pergunta sobre estrutura/pastas em si (não conteúdo) — usar `ls`/`find` direto.
- Precisa do texto exato/grep de um termo específico (ID, nome de arquivo, string literal) —
  `grep -r` é mais preciso que busca semântica para isso.
- Índice pode estar desatualizado após edição recente no projeto fonte — rodar
  `recall ingest --project <nome>` (ou `--all`) para reindexar (idempotente, seguro repetir).

## Pré-requisito

Qdrant local precisa estar de pé (`recall search`/`recall ingest` sobem sozinhos via
`podman compose up -d` se a porta 6333 não responder). Se um projeto ainda não está configurado
no `recall.toml`, ele não aparece na busca — configurar é fora do escopo desta skill.
