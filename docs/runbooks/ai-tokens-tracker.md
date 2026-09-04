# Runbook: ai-tokens-tracker — consumo de token de agentes de IA

Repositório: [`github.com/goriok/ai-tokens-tracker`](https://github.com/goriok/ai-tokens-tracker)

Hoje cobre só o `agy` (Google Antigravity CLI) — outros agentes estão planejados, sem data
definida. Não depende de `recall` nem deste repositório.

## Instalar

```bash
git clone git@github.com:goriok/ai-tokens-tracker.git
cd ai-tokens-tracker
bash install.sh
```

Symlinka `agystatus`, `agysnapshot`, `agywidget`, `agydelegate` em `~/.local/bin/`.

## Instalar como plugin (Claude Code ou Antigravity)

```
# Claude Code
/plugin marketplace add goriok/ai-tokens-tracker
/plugin install ai-tokens-tracker
```

```bash
# Antigravity
agy plugin install ./ai-tokens-tracker/plugins/ai-tokens-tracker
```

## Registrar o primeiro snapshot de quota

```bash
agysnapshot
```

Roda `agy -p "/usage" --output-format json` — **custo zero de token**, é uma consulta de estado
da conta, não uma chamada de modelo. Grava a quota semanal restante por grupo de modelo
(Gemini vs. Claude/GPT) no SQLite local (`~/.local/share/ai-tokens-tracker/usage.db`).

## Agendar coleta automática

```bash
crontab -e
# adicionar:
0 * * * * /home/$USER/.local/bin/agysnapshot >> /tmp/agysnapshot.log 2>&1
```

De hora em hora é suficiente — a quota é semanal, alta frequência não agrega dado útil.

## Ver o relatório

```bash
agystatus
```

Gera e abre um HTML standalone (gráficos via Chart.js, sem servidor) com a quota ao longo do
tempo e as chamadas rastreadas via `agy-track.py`/`agydelegate`.

## Delegar uma tarefa com escolha automática de modelo

```bash
agydelegate --complexity low --task "revisar changelog" "revise este texto: ..."
```

`--complexity` (`low`/`medium`/`high`) cruza com a quota mais recente registrada — se o grupo
de modelo preferido para aquela complexidade estiver com quota baixa, cai para o outro grupo
automaticamente. Rodar `agysnapshot` antes garante que a decisão usa dado fresco (o comando não
consulta `/usage` sozinho, para manter uma única chamada real de modelo por invocação).

## Troubleshooting

| Sintoma | Causa provável | Ação |
|---|---|---|
| `agysnapshot` falha com "No usage groups in response" | Formato de saída do `agy` mudou | Rodar `agy -p "/usage" --output-format json` manualmente e comparar com o schema esperado em `AgyCliRunner.fetch_usage` |
| Widget GTK não mostra dado | Nenhum snapshot registrado ainda | Rodar `agysnapshot` manualmente uma vez |
| `agystatus` mostra relatório vazio | `$AGY_TOOL_DB` apontando para um arquivo diferente do usado por `agysnapshot`/`agydelegate` | Conferir a variável de ambiente nos dois contextos |
