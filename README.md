# eng-ctx-skills

Skills para criar e manter repositórios de contexto de engenharia (padrão `ctx-*`): estrutura
canônica de pastas, tópicos, campo `Status`, MADRs, prompts para agent-tools externas e busca
semântica sobre o conteúdo indexado.

## Skills

| Skill | O que faz |
|---|---|
| `ctx-repo-init` | Inicializa a estrutura canônica de um repo `ctx-*` novo — pastas, `CLAUDE.md`, `AGENTS.md`, `CHANGELOG.md`. |
| `ctx-repo-check` | Audita um repo `ctx-*` existente contra o padrão canônico e reporta divergências. |
| `ctx-repo-fix` | Aplica as correções apontadas pelo `ctx-repo-check`. |
| `ctx-repo-topic-add` | Adiciona um tópico novo a um repo `ctx-*` existente. |
| `ctx-repo-promptl` | Cria/atualiza um prompt para agent-tools externas (Gemini Gem, NotebookLM, GPT) a partir do contexto do repo. |
| `ctx-recall` | Busca semântica local via `recall` (Qdrant + Ollama) sobre qualquer projeto/documentação indexada, incluindo repos `ctx-*`. |

## Instalação no Claude Code

Este repositório é um plugin marketplace (`.claude-plugin/marketplace.json`). Requer acesso Git autenticado ao repositório.

```
/plugin marketplace add goriok/eng-ctx-skills
/plugin install eng-ctx-skills@eng-ctx-skills
```

Alternativa via `settings.json` (global ou de projeto):

```json
{
  "extraKnownMarketplaces": {
    "eng-ctx-skills": {
      "source": {
        "source": "github",
        "repo": "goriok/eng-ctx-skills"
      }
    }
  },
  "enabledPlugins": {
    "eng-ctx-skills@eng-ctx-skills": true
  }
}
```

**Atualizar:** `/plugin marketplace update goriok/eng-ctx-skills` puxa o marketplace mais recente,
seguido de `/reload-plugins` para o Claude Code recarregar as skills já instaladas com o conteúdo
novo — sem o reload, o autocomplete de slash command continua mostrando a versão antiga em cache.

## Instalação no Hermes

O Hermes carrega Agent Skills no mesmo formato (`SKILL.md`). Instale direto do repositório:

```bash
hermes plugins install goriok/eng-ctx-skills --no-enable
hermes plugins enable eng-ctx-skills
```

As skills ficam disponíveis em `~/.hermes/skills/` e cada uma vira um slash command automaticamente.

**Atualizar:** `hermes plugins update eng-ctx-skills` (ou `hermes plugins install goriok/eng-ctx-skills`
de novo, que reinstala por cima).

## Instalação no Antigravity (`agy`)

```bash
git clone git@github.com:goriok/eng-ctx-skills.git
agy plugin install ./eng-ctx-skills/plugins/eng-ctx-skills
```

`plugins/eng-ctx-skills/` é a pasta de plugin nativa do `agy` neste repositório: `plugin.json` +
um symlink `skills/` apontando para `../../skills` (a mesma pasta que o Claude Code usa) — uma
única fonte de skills, sem duplicar conteúdo entre os dois formatos. `agy plugin install` copia
esse registro para a config local do `agy` (`~/.gemini/config/plugins/eng-ctx-skills/`) — não é
um link vivo para o clone; editar o clone depois não tem efeito até reinstalar.

**Atualizar:** `git pull` no clone, depois `agy plugin install ./eng-ctx-skills/plugins/eng-ctx-skills`
de novo — sobrescreve o registro anterior. Não existe `agy plugin update`. Sem esse passo, uma
skill renomeada ou com conteúdo alterado no repositório continua desatualizada no `agy`
indefinidamente — ele não detecta a mudança sozinho. `agy plugin validate
./eng-ctx-skills/plugins/eng-ctx-skills` confirma que a pasta está bem formada antes de instalar.

## Relação com `my-skills`

As skills `ctx-repo-*` foram movidas do repositório pessoal `goriok/my-skills` (privado) para cá —
não existem mais duplicadas nos dois lugares. `ctx-recall` é exceção deliberada: existe também
como `recall-search` em `my-skills`, com escopo mais genérico (qualquer projeto indexado); as duas
evoluem separadamente a partir daqui.
