# Ferramental de apoio à estrutura de repositório de contexto

Três peças operam sobre a convenção descrita em [estrutura.md](estrutura.md). Cada uma é
independente das outras — nenhuma depende de as demais estarem instaladas.

| Peça | Repositório | Papel |
|---|---|---|
| **eng-ctx-skills** | este repositório | Skills `ctx-repo-*` que operam a convenção estrutural (inicializar, auditar, corrigir, adicionar tópico), mais `ctx-recall` |
| **recall** | [`github.com/goriok/recall`](https://github.com/goriok/recall) | Busca semântica local (Qdrant embutido + Ollama) sobre o Markdown de qualquer repositório de contexto configurado |
| **ai-tokens-tracker** | [`github.com/goriok/ai-tokens-tracker`](https://github.com/goriok/ai-tokens-tracker) | Rastreamento de consumo de token/quota de agentes de IA, com delegação de tarefas por complexidade |

Passo a passo de instalação e uso de cada uma em [runbooks/](runbooks/).

## recall: busca semântica, agnóstica da convenção

`recall` indexa arquivos Markdown num Qdrant local e expõe busca via CLI ou MCP. É agnóstico da
convenção `ctx-*` — não sabe o que é campo `Status`, nem distingue `human/` de `ai/`. Na prática:

- Um resultado de busca nunca deve ser tratado como `approved` só por ter aparecido — é preciso
  abrir o arquivo fonte e checar o `Status` antes de usar como base de geração.
- `madrs/` normalmente fica fora do índice — decisões arquiteturais raramente aparecem em busca
  semântica geral, e o volume por tópico costuma ser baixo o suficiente para leitura direta.

## ai-tokens-tracker: consumo de token como dado

Mede consumo de agentes de IA (hoje só Google Antigravity CLI, `agy`) sem indexar conteúdo — sem
relação direta com a convenção estrutural. A conexão é indireta: uma baseline de consumo por
estratégia (ex. "gerar um tópico novo com `ctx-repo-topic-add` custa X tokens em média") só é
possível calculando os dois em conjunto.

## Como as três peças interagem

```text
repositório de contexto
    │
    ├─ recall indexa Markdown → busca semântica (menos token gasto localizando conteúdo)
    │
    └─ skills ctx-repo-* (este repositório) operam sobre a estrutura (Status, topics/, MADR)
           │
           └─ rodam dentro de um agente (Claude Code, Antigravity/agy)
                  │
                  └─ ai-tokens-tracker mede o consumo de token desse agente
```
