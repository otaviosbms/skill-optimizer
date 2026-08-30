# Catálogo de Anti-Patterns — Skill Optimizer v5

Referência para a Fase 1 (Audit). Para cada ocorrência encontrada, registre: `[TIPO]`, arquivo + linha/seção, tokens desperdiçados estimados.

---

## [FAKE-OFFLOADING]
**Definição:** O prompt delega para script externo, mas descreve no Markdown as regras de validação, parsing ou lógica da tarefa — forçando o LLM a executar mentalmente a mesma tarefa que seria do script.
**Sinal:** Verbo "use o script X" ou "execute Y" seguido de instruções detalhadas de como a lógica funciona.
**Correção:** O script deve ser a única fonte de verdade. Remova a descrição da lógica do prompt; mantenha apenas a invocação.

---

## [PROSE-CONDITIONAL]
**Definição:** Fluxos condicionais complexos escritos em prosa corrida ("Se X acontecer... senão se Y... mas se Z...").
**Sinal:** Blocos de texto com 3+ condições encadeadas sem estrutura de tabela ou lista de decisão.
**Correção:** Substitua por tabela de decisão compacta ou extraia a lógica para script.

---

## [EMBEDDED-REGEX]
**Definição:** Expressões regulares ou patterns de parsing embutidos diretamente no corpo do prompt.
**Sinal:** Presença de `\d+`, `[A-Z]+`, `(?:...)`, `^.*$` ou similares no texto Markdown.
**Correção:** Mova para arquivo de configuração em `data/` ou implemente em script.

---

## [FEW-SHOT-INFLATION]
**Definição:** Exemplos excessivos que consomem tokens sem ganho marginal de aprendizado.
**Sinal:** 3+ exemplos cobrindo o mesmo padrão comportamental.
**Correção:** Mantenha 1-2 exemplos canônicos. Mova os demais para `data/examples/` e referencie quando necessário.

---

## [REDUNDANT-RULES]
**Definição:** A mesma instrução expressa múltiplas vezes com palavras diferentes em seções distintas.
**Sinal:** Dois ou mais parágrafos/itens com semântica equivalente ao analisar o intent real.
**Correção:** Unifique em uma única instrução precisa na posição de maior impacto contextual.

---

## [STATIC-KNOWLEDGE-INLINE]
**Definição:** Dados de referência estáticos (tabelas de valores, schemas, catálogos) embutidos no prompt, recarregados em todo activation.
**Sinal:** Tabelas Markdown, listas de enumeração ou blocos de dados que não mudam entre execuções.
**Correção:** Mova para `data/` e leia via `Read` tool apenas quando a sub-tarefa exigir (lazy loading).

---

## [ALGORITHMIC-LLM]
**Definição:** O LLM é instruído a executar um algoritmo sequencial passo a passo em vez de delegar para código.
**Sinal:** Sequências do tipo "1. Parse X, 2. Valide Y, 3. Calcule Z, 4. Formate W" onde cada passo é determinístico.
**Correção:** Implemente o algoritmo em script procedural. O LLM invoca o script e interpreta o resultado.

---

## [VERBOSE-OUTPUT-TEMPLATE]
**Definição:** O formato de saída é definido por um template de placeholder com muitas linhas inline no prompt, recarregado a cada execução, em vez de um schema compacto.
**Sinal:** Blocos de output com 15+ linhas contendo `[PLACEHOLDER]`, valores de exemplo ou estruturas repetitivas que descrevem o formato ao invés de prescrevê-lo de forma enxuta.
**Correção:** Substitua por um schema mínimo com os campos obrigatórios. Mova exemplos de output para `data/examples/` se necessário.

---

## [CONTEXT-REINJECTION]
**Definição:** A skill instrui o LLM a reenviar dados que ele mesmo produziu na mesma sessão para reprocessamento ou confirmação — gastando tokens em dados já disponíveis no contexto.
**Sinal:** Instruções como "com base no relatório acima, re-analise...", "repita o diagnóstico considerando...", ou passos que pedem para reler output já gerado.
**Correção:** Referencie outputs anteriores diretamente no raciocínio sem reinjeção. Se validação é necessária, faça inline no mesmo passo, não em uma etapa separada de releitura.

---

## [SCOPE-CREEP-INSTRUCTIONS]
**Definição:** A skill inclui instruções fora do seu escopo central declarado, inflando o prompt com comportamentos que não servem ao propósito da skill e ativam em toda invocação.
**Sinal:** Seções que descrevem tarefas não relacionadas ao objetivo principal (ex: uma skill de geração de código que também instrui sobre processo de PR, comunicação com o time ou políticas de deploy).
**Correção:** Remova toda instrução cujo propósito não é diretamente necessário para entregar o resultado principal da skill. Redirecione responsabilidades periféricas para outras skills especializadas.

---

## [EXPLORATORY-ROUNDS]
**Definição:** A skill usa verbos ou instruções ambíguas que induzem o modelo a rodar scripts/passos exploratórios intermediários para "entender" os dados antes de agir — mesmo quando os dados já são suficientes para ação direta. O padrão se repete em qualquer fase que receba dados estruturados: coleta via API, leitura de JSON, agregação de resultados.
**Sinal:** Instruções como "analise cada jornada", "inspecione os dados", "revise os resultados" sem especificar que a análise deve ocorrer diretamente no script de processamento. Ausência de uma diretriz explícita proibindo rounds exploratórios. Passos que recebem dados estruturados de um script anterior mas não especificam "vá direto para X sem inspeção intermediária".
**Exemplo real (fase NR):** Instrução "analisar cada jornada no chat" com 833 registros induziu o modelo a rodar 4 scripts Python de amostragem (~12k tokens) antes de escrever o script de classificação — mesmo tendo os dados suficientes desde o primeiro output.
**Exemplo real (fase JSON):** Mesmo com o `[EXPLORATORY-ROUNDS]` proibido para o nr_client, o modelo rodou 5 scripts Python de agregação/deep-dive no JSON resultante (~3–4k tokens extras) antes de escrever o classificador final — porque a diretriz só cobria a fase de coleta, não a de análise.
**Correção:** (1) Substitua verbos ambíguos por ações diretas: em vez de "analise os dados", escreva "aplique as regras de classificação de `data/X.md` diretamente sobre o JSON — sem scripts exploratórios intermediários". (2) A diretriz anti-exploratória deve cobrir **todas** as fases da skill, não apenas a coleta: "Não execute scripts de inspeção ou amostragem em nenhuma etapa. Se os dados chegaram estruturados, vá direto para o processamento final." (3) Especifique o formato de saída esperado de cada passo para que o modelo saiba exatamente o que fazer com o input sem precisar explorar.

---

## [READ-MODIFY-REWRITE]
**Definição:** A skill instrui o LLM a fazer `Read` de um arquivo template estático (JS, HTML, config), modificar 1–3 placeholders e reescrever o arquivo inteiro — consumindo os tokens do template duas vezes: uma na leitura (input de contexto) e outra na escrita via heredoc Bash ou Write tool (output como tool call).
**Sinal:** Instrução do tipo "Read `data/X` → substituir `A`, `B`, `C` → escrever `/tmp/X`" onde o arquivo tem mais de 50 linhas e menos de 10% do conteúdo muda entre execuções. Arquivos de template com lógica de renderização misturada aos dados de saída.
**Exemplo real:** `data/docx-template.js` tem 352 linhas; a instrução causou Read (352 linhas no contexto) + Write de cópia modificada (400 linhas como heredoc Bash) → ~750 linhas de código de template consumidas como tokens para alterar apenas 3 variáveis (`atendimentos[]`, `resumo_executivo`, `OUTPUT_PATH`).
**Correção:** Separe renderização de dados no template: o arquivo estático em `data/` deve `require('/tmp/relatorio_data.js')` e ser copiado com um simples `cp` (sem Read, sem Write). O LLM gera apenas o arquivo de dados pequeno (`/tmp/relatorio_data.js`); o template pesado nunca entra no contexto.

---

## [UNBOUNDED-CONTEXT-ACCUMULATION]
**Definição:** A skill instrui o LLM a processar múltiplas unidades de trabalho (módulos, arquivos, domínios) em sequência no mesmo contexto, sem isolar leituras entre iterações. Arquivos grandes lidos na iteração 1 permanecem acumulados no contexto da iteração N, inflando o custo total de forma proporcional ao número de unidades × tamanho médio dos arquivos.
**Sinal:** Instrução do tipo "repita A→E para cada módulo" ou "processe cada X em sequência" **combinada com** a skill operando sobre repos ou domínios de negócio cujos arquivos típicos têm >300 linhas (estratégias, resolvers, services, handlers). Skills que operam sobre arquivos pequenos (<100 linhas típicos) não configuram o anti-pattern — o overhead do isolamento superaria o benefício.
**Exemplo real:** Skill `pedbot-code-docs` documentando `maria-gpt-api`: `sale-chat-gpt.strategy.ts` (7000+ linhas) + `conversation.resolver.ts` (1000 linhas) lidos nos primeiros módulos permaneceram no contexto dos módulos seguintes, resultando em ~37k tokens totais para uma execução incremental de 9 módulos.
**Correção:** Se os arquivos típicos do alvo têm >300 linhas por módulo, instrua o uso de subagent por módulo: cada subagent parte do zero, lê apenas os arquivos da sua unidade de trabalho e retorna o resultado (chunks, análise, output). O contexto principal recebe apenas os resultados, não o conteúdo dos arquivos. Se o alvo tem arquivos <100 linhas, o loop sequencial é aceitável — mantenha como está.

---

## [BRITTLE-EXTERNAL-SCRIPT]
**Definição:** A skill delega processamento em lote para um script externo (`scripts/`), mas o script gerado não possui tratamento de erro em chamadas externas, não persiste progresso incrementalmente e não paraleliza quando opera sobre N > 100 registros — tornando qualquer falha transitória catastrófica (perda total do progresso) e o tempo de execução linear e lento.
**Sinal:** Script que (a) chama API/rede sem retry ou try/except; (b) escreve output apenas ao final, sem checkpoint intermediário; (c) processa registros sequencialmente em loop simples quando o volume é alto. Qualquer combinação de dois desses três indícios já configura o anti-pattern.
**Exemplo real:** Script `nr_client.py` com 833 registros falhou no registro 290 por resposta `null` da API do New Relic — sem retry, perdeu todo o progresso e foi necessário reiniciar do zero. O processamento sequencial demandava ~15-20 minutos para completar.
**Correção:** Para scripts que processam N > 100 registros via rede, o SKILL.md deve exigir explicitamente três propriedades no script gerado: (1) **Retry com backoff** em toda chamada externa (`for attempt in range(3): ... time.sleep(2**attempt)`); (2) **Checkpoint incremental** — salvar resultados a cada registro ou em lotes (ex: `json.dump` linha a linha em NDJSON, ou salvar parcial a cada 50 registros); (3) **Paralelismo controlado** — usar `ThreadPoolExecutor(max_workers=5)` ou equivalente para I/O-bound workloads, com limite explícito de workers para não sobrecarregar a API. Adicione ao SKILL.md uma seção "Requisitos do script de coleta" listando essas três propriedades como obrigatórias.
