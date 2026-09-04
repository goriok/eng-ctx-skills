# Runbook: recall — busca semântica sobre repositórios de contexto

Repositório: [`github.com/goriok/recall`](https://github.com/goriok/recall)

## Instalar

```bash
git clone git@github.com:goriok/recall.git
cd recall
./bootstrap.sh
```

Instala o CLI (`recall`, `recall-mcp`), baixa o modelo de embedding (`nomic-embed-text` via
Ollama), e copia `~/.config/recall/recall.toml` se ainda não existir.

**Pré-requisito:** [Ollama](https://ollama.com/download) instalado e rodando.

## Instalar como plugin (Claude Code)

```
/plugin marketplace add goriok/recall
/plugin install recall
```

Registra `recall-mcp` via `uvx` — não precisa clonar nem rodar `bootstrap.sh` neste modo.

## Configurar uma entrada por tópico

Editar `~/.config/recall/recall.toml`, uma entrada `[[projects]]` por tópico do repositório de
contexto:

```toml
[[projects]]
name = "<repo>-<topico>"
path = "<caminho-absoluto>/topics/<topico>"
collection = "<repo>-<topico>"
glob = "**/*.md"
path_exclude = [".git", ".workspace", "madrs", "_templates"]
```

`madrs/` fica de fora do índice — decisões arquiteturais raramente aparecem em busca semântica
geral, e o volume por tópico é baixo o suficiente para leitura direta quando precisar.

## Indexar

```bash
recall ingest --project <nome>
```

Cada chunk tem um ID determinístico (`sha256(source::heading::index)`), e a indexação faz upsert
por esse ID — reindexar depois de editar só o texto de uma seção existente (mesmo heading, mesma
posição) sobrescreve o chunk antigo, sem duplicar.

Isso não cobre cortar ou renomear uma seção: o heading muda a chave do ID, então o chunk antigo
não é sobrescrito nem removido — fica órfão na collection indefinidamente. Depois de uma edição
estrutural (não só troca de texto dentro de uma seção), rodar `recall ingest --recreate --project
<nome>` para dropar e reconstruir a collection do zero.

## Buscar

```bash
recall search "<pergunta>" --in <nome>
```

Cada resultado retorna o `source` (path absoluto do arquivo) e o trecho do chunk. **Sempre abrir
o arquivo fonte antes de citar o resultado como base de geração** — o chunk não carrega o campo
`Status`, então a confiabilidade do conteúdo precisa ser checada diretamente no arquivo.

`--min-score <0-1>` descarta resultados abaixo dessa similaridade (score de cosseno do Qdrant,
mostrado em cada resultado). Sem essa flag, `recall search` sempre devolve os `--top` mais
próximos, mesmo que nenhum seja realmente relevante — útil para saber se a base tem algo sobre o
assunto ou não, mas ruim quando se quer só resultado de alta confiança. Não há um valor padrão
certo; começar sem a flag, olhar o score exibido, e subir o corte (ex.: `0.6`-`0.7`) se o
resultado top vier abaixo disso e ainda assim parecer pouco relacionado.

## Troubleshooting

| Sintoma | Causa provável | Ação |
|---|---|---|
| `recall search` não acha nada com `--in <nome>` | Nome de collection errado — não gera erro, retorna vazio silenciosamente | `recall collections list` para conferir o nome exato |
| Resultado desatualizado após editar um doc | Índice não foi atualizado | `recall ingest --project <nome>` de novo |
| `recall`/`recall-mcp` trava tentando subir Qdrant | Só ocorre em modo servidor (`host`/`port` configurado) — modo embutido (padrão) não depende de nenhum processo externo | Confirmar que `[qdrant]` no `recall.toml` não tem `host` definido |
