# Rubrica de Skill Maturity Score (0-100)

Some os pontos de cada eixo. O total é o Maturity Score final.

> **Nota de calibração:** Os percentuais do Eixo 1 são relativos à skill. Para skills muito pequenas (baseline < 1K tokens), use tokens absolutos economizados como critério complementar: economia > 500 tokens = equivalente a ">40%"; economia > 200 tokens = equivalente a ">20%".

---

## Eixo 1 — Eficiência de Tokens no Contexto Ativo (40 pts)

Considere apenas os tokens carregados automaticamente a cada execução (sem lazy loading).

| Redução alcançada | Pontos |
|-------------------|--------|
| > 60%             | 40     |
| 40 – 60%          | 30     |
| 20 – 40%          | 20     |
| < 20%             | 10     |
| Sem redução       | 0      |

---

## Eixo 2 — Ausência de Anti-Patterns (30 pts)

| Anti-patterns detectados | Pontos |
|--------------------------|--------|
| 0                        | 30     |
| 1                        | 20     |
| 2                        | 10     |
| 3 ou mais                | 0      |

---

## Eixo 3 — Arquitetura de Offloading (20 pts)

| Critério | Pontos |
|----------|--------|
| Lógica procedural extraída para `scripts/` | +10 |
| Dados estáticos extraídos para `data/` com lazy loading | +5 |
| Sub-tarefas identificadas como candidatas a model offload | +5 |

> **Skills conversacionais:** Se a skill não contém lógica procedural algorítmica, o critério `scripts/` não se aplica e os +10 pts são automaticamente concedidos — a ausência de scripts é a decisão correta para o tipo de skill, não uma omissão. Documente explicitamente no relatório: "Nenhuma lógica procedural identificada — critério scripts/ N/A, 10 pts concedidos."

---

## Eixo 4 — Preservação de Qualidade — Quality Gate (10 pts)

| Resultado do Quality Gate (Fase 3) | Pontos |
|------------------------------------|--------|
| 4/4 checks aprovados               | 10     |
| 3/4 checks aprovados               | 5      |
| 2/4 ou menos aprovados             | 0      |

---

## Interpretação do Score Final

| Faixa  | Diagnóstico                                          |
|--------|------------------------------------------------------|
| 80–100 | Pronto para produção                                 |
| 60–79  | Bom — refinamentos pontuais recomendados             |
| 40–59  | Funcional — oportunidades significativas de melhoria |
| 0–39   | Requer refatoração profunda antes de uso             |
