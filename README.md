# skill-optimizer

Skill do [Claude Code](https://claude.com/claude-code) que audita e refatora **outras** skills para máxima eficiência de tokens, preservando 100% das funcionalidades e da qualidade de resposta originais.

## O que ela faz

Dada uma skill alvo, executa um pipeline em três fases:

1. **Audit** — mapeia toda a estrutura da skill (SKILL.md, `data/`, `scripts/`), estima o custo de ativação em tokens e detecta anti-patterns catalogados em [`skill-optimizer/data/anti-patterns.md`](skill-optimizer/data/anti-patterns.md) (ex: lógica algorítmica descrita em prosa, dados estáticos inline, exemplos redundantes, scripts sem retry/checkpoint).
2. **Refactor** — produz uma versão `[skill-original]-optimized/`, movendo dados estáticos para `data/` (lazy loading via `Read`), lógica procedural para `scripts/`, e removendo instruções sem impacto mensurável.
3. **Quality Gate** — confirma correspondência 1:1 entre sub-tarefas do original e da versão otimizada antes de reportar conclusão, cobrindo funcionalidade, cobertura de casos de uso, tom/persona e regras críticas.

O resultado inclui um relatório de auditoria com economia estimada de tokens e um **Maturity Score (0–100)**, calculado pelos quatro eixos definidos em [`skill-optimizer/data/rubric.md`](skill-optimizer/data/rubric.md): eficiência de tokens, ausência de anti-patterns, arquitetura de offloading e preservação de qualidade.

## Estrutura do repositório

```text
skill-optimizer/          # pacote instalável da skill
├── SKILL.md              # definição da skill (role, pipeline, output)
└── data/
    ├── anti-patterns.md  # catálogo de anti-patterns para a fase de Audit
    └── rubric.md          # rubrica do Maturity Score
```

O conteúdo de `skill-optimizer/` é exatamente o que precisa existir em `~/.claude/skills/skill-optimizer/` para a skill funcionar — os arquivos na raiz do repositório (README, licença, etc.) são apenas metadados do projeto e não são lidos pelo Claude Code.

## Instalação

Clone (ou copie) a pasta `skill-optimizer/` para dentro de `~/.claude/skills/`:

```bash
git clone https://github.com/otaviosbms/skill-optimizer.git /tmp/skill-optimizer-repo
cp -r /tmp/skill-optimizer-repo/skill-optimizer ~/.claude/skills/skill-optimizer
```

Ou, se preferir manter o clone como fonte e refletir atualizações automaticamente, crie um symlink:

```bash
ln -s /caminho/para/skill-optimizer-repo/skill-optimizer ~/.claude/skills/skill-optimizer
```

## Uso

No Claude Code, invoque a skill informando a skill alvo a ser otimizada (por nome, se estiver em `~/.claude/skills/`, ou por path completo). A skill localiza os arquivos, executa o pipeline Audit → Refactor → Quality Gate e cria a versão otimizada em disco.
