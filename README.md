# Guia prático de criação de skills

Um processo em seis fases (da 0 à 5) para criar skills de agente que **realmente melhoram o resultado**, usando duas ferramentas, cada uma no papel em que é mais forte:

| Ferramenta | Papel no processo | Fases |
|------------|-------------------|-------|
| [`skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) (Anthropic) | **Estruturar e medir:** entende a intenção, entrevista, redige o `SKILL.md`, roda evals, compara versões e otimiza a `description` | 2 e 4 |
| [`writing-for-agents`](https://github.com/mattpocock/skills/tree/main/skills/productivity/writing-for-agents) (Matt Pocock) | **Lapidar:** ensina a escrever documentos para agentes (ponteiros de contexto, hierarquia da informação, critérios de conclusão, poda) | 3 |

> **Ideia central:** o gerador produz um rascunho **completo, porém inchado**; o filtro de poda o transforma em algo **enxuto e previsível**; o teste com vs. sem skill prova que valeu a pena.

### Como ler as etiquetas de origem

Cada prática traz a origem entre parênteses, para você saber o quanto confiar nela:

- **(SC)**: está no `skill-creator`.
- **(WFA)**: está no `writing-for-agents`.
- **(autor)**: recomendação deste guia, **não** presente nas duas fontes. Vem de boas práticas gerais de engenharia e da documentação da Anthropic como lembrada pelo autor, sem verificação linha a linha. Trate como sugestão a validar no seu contexto.

Nada aqui se apoia em pesquisa acadêmica. As fontes são documentos de instrução escritos por praticantes.

## Visão do processo

![Fluxo do processo de criação de skills](docs/fluxo-skills.png)

---

## Fase 0: Definir (antes de tudo)

Responda por escrito, em poucas linhas:

1. **Objetivo:** o que a skill permite ao agente fazer, em uma frase. (SC)
2. **Gatilho:** quando ela deve ser acionada e com quais frases do usuário. (SC)
3. **Saída esperada:** formato e critério de sucesso verificável. (SC)
4. **Não-objetivos:** o que a skill deliberadamente **não** faz. Uma skill, uma responsabilidade. (autor)
5. **Quem aciona:** o modelo sozinho ou só você, digitando o nome? Se só você, use `disable-model-invocation: true` e pague **zero** de contexto; se o modelo precisa alcançá-la, a `description` fica sempre carregada. (WFA)
6. **O artefato certo:** se a regra é mecânica e deve valer sempre, prefira hook, linter ou CI; se é uma convenção curta do projeto, `CLAUDE.md`/`AGENTS.md`; se é acesso a sistema externo, MCP. Skill é para **processo e julgamento** que o agente precisa seguir sob demanda. (autor)

**Entregável:** um parágrafo com objetivo, gatilho, saída esperada e não-objetivos.

## Fase 1: Preparar

1. **Escolha 3 cenários reais** em que a IA erra, varia ou esquece algo. Inclua 1 caso de borda. (autor)
2. **Rode cada cenário sem skill**, em sessão nova, e guarde as saídas. É o seu baseline. (SC: o baseline "sem skill" é a referência de toda a avaliação)
3. **Reúna o material-fonte**, se existir: documentação, histórico de uma tarefa bem-sucedida, regras internas. Se a conversa atual já contém o fluxo que você quer capturar, o `skill-creator` extrai dela as ferramentas usadas, a sequência e as correções que você fez. (SC)
4. **Decida:** se o baseline já sai bom, pare aqui. A skill não se justifica. (autor)

Para saídas **subjetivas** (estilo de escrita, design), testes quantitativos costumam não valer o esforço; prefira revisão humana. (SC)

**Entregável:** `cenarios.md` (3 pedidos) e `baseline/` (saídas sem skill).

## Fase 2: Estruturar com `skill-creator`

Peça ao `skill-creator` que conduza as etapas dele: **entender a intenção**, **entrevistar** (casos de borda, formatos de entrada e saída, arquivos de exemplo, critérios de sucesso, dependências) e **redigir o rascunho**. Com material-fonte, ele parte dele; sem material, a entrevista faz o papel de levantar requisitos. (SC)

```text
Use a skill skill-creator. Quero uma skill para [tarefa].
Material-fonte: [anexo, ou "nenhum"].
Objetivo, gatilho e não-objetivos estão na Fase 0 (abaixo).
Entreviste-me sobre casos de borda e critérios de sucesso antes de escrever.
Coloque o que for volumoso em references/ e marque o que é operação
determinística (candidata a script).
```

Anatomia de uma skill: `SKILL.md` (obrigatório) mais, se precisar, `scripts/` (código determinístico), `references/` (documentos lidos sob demanda) e `assets/` (arquivos usados na saída, como templates). (SC)

**Entregável:** rascunho de `SKILL.md`. Esperado: longo e redundante. É normal.

## Fase 3: Lapidar com `writing-for-agents`

O rascunho vira algo enxuto aplicando as ideias abaixo. Peça à skill que revise o arquivo, ou use esta seção como roteiro.

### 3.1 A `description` é um ponteiro de contexto (WFA)

É a única parte da skill que o agente vê sempre, então cada palavra custa em **todo** turno e o texto decide **quando** a skill é alcançada.

- Diga **o que a skill faz e quando usar**. Todo gatilho fica aqui, não no corpo. (SC)
- Coloque a palavra mais importante **no início**. (WFA)
- **Um gatilho por ramo.** Sinônimos que renomeiam o mesmo caso são um ramo escrito duas vezes: junte. (WFA)
- Corte o que o corpo já diz. (WFA)
- O Claude tende a **subacionar** skills; uma descrição um pouco mais assertiva compensa, desde que cada gatilho seja distinto. Valide com o teste de acionamento da Fase 4. (SC)
- Se o gatilho está mal escrito, **melhore a redação** antes de copiar o conteúdo alvo para dentro do corpo. (WFA)
- Escreva em terceira pessoa. (autor)

### 3.2 Hierarquia da informação e divulgação progressiva

Cada conteúdo ocupa um degrau: **passo no arquivo** (o que o agente faz, em ordem), **referência no arquivo** (consultada sob demanda) ou **referência em outro arquivo**, alcançada por um ponteiro. (WFA)

- Três níveis de carga: metadados sempre no contexto; corpo do `SKILL.md` quando a skill dispara (idealmente abaixo de 500 linhas); recursos em `references/` e `scripts/` só quando necessários. (SC)
- Critério de ramo: deixe no corpo o que **todo** ramo precisa; mova para `references/` o que só **alguns** ramos usam, deixando no corpo um ponteiro que diz **quando** ler. (WFA)
- Referências com mais de 300 linhas levam sumário. Quando a skill atende vários domínios, organize um arquivo por variante (ex.: `aws.md`, `gcp.md`). (SC)
- **Co-localização:** definição, regras e ressalvas de um conceito ficam sob o mesmo título, não espalhadas. (WFA)
- Cuidado com o **sprawl**: documento longo demais dilui a atenção mesmo com todas as linhas úteis. A cura é descer material para `references/`, não cortar às cegas. (WFA)

### 3.3 Passos terminam em um critério de conclusão (WFA)

Todo passo precisa dizer **quando está pronto**.

- **Clareza:** o agente consegue distinguir "feito" de "não feito"? Critério vago ("entendimento alcançado") convida à **conclusão prematura**.
- **Exigência:** "todo modelo modificado listado" força trabalho completo; "produza uma lista de mudanças" não.
- Os melhores critérios são verificáveis e exaustivos. Afie o critério primeiro; só divida a sequência em duas se ele for irredutivelmente vago **e** você observar o agente apressando.

### 3.4 Como escrever as instruções

**Faça:**
- **Imperativo** e **explique o porquê** de cada regra não óbvia. O modelo generaliza melhor com a razão do que com uma ordem seca. (SC)
- **Descreva o comportamento desejado** ("escreva comentários de uma linha"), em vez de proibir o indesejado. (WFA)
- Use **uma palavra que o modelo já conhece** (uma *leading word*, como *red*, que transforma um critério difuso em estado binário: o loop fica *red* no bug ou não fica) no lugar de frases repetidas. Reaproveite o termo como token, nunca como explicação. (WFA)
- **Generalize:** a skill será usada em pedidos que você não previu. Não ajuste só para os exemplos de teste. (SC)
- Mantenha **terminologia consistente** (um nome por conceito). (autor)
- Dê **um padrão por decisão**, não um menu de opções. (autor)

**Evite:**
- MUST, ALWAYS e NEVER em caixa alta: é sinal amarelo. Reformule explicando o motivo. (SC)
- **Proibições sozinhas.** Proibir arrasta o comportamento proibido para o contexto. Use a proibição só como guarda-corpo que você não consegue escrever de forma positiva, e **sempre pareada** com o comportamento-alvo. (WFA)
- **Repetir o mesmo significado** em dois lugares. Mantenha cada regra em uma **fonte única da verdade**. (WFA)
- **Copiar o ambiente.** Scripts do `package.json`, arquivos de config e `--help` já são fonte da verdade; o documento só guarda o que o agente não encontra olhando: a convenção não escrita, o porquê de uma escolha, a armadilha. (WFA)

### 3.5 Poda: o teste *no-op* (WFA, SC)

Para cada frase: se removê-la **não muda o comportamento** do agente, apague a frase inteira. O teste depende do modelo (o que é no-op para um não é para outro), então decida **rodando**, não debatendo. Leia também as **transcrições** das execuções: partes da skill que fazem o modelo perder tempo são candidatas a corte. (SC)

Sem disciplina de poda o destino é o **sedimento**: camadas velhas que ficam porque acrescentar parece seguro e remover parece arriscado. Prefira documentos curtos; são mais fáceis de manter relevantes. (WFA)

### 3.6 Estrutura sugerida para o corpo do `SKILL.md` (autor)

```markdown
---
name: nome-da-skill
description: O que faz. Use quando [gatilhos distintos].
---

# Nome da skill

## Quando usar / quando não usar
## Fluxo            (passos, cada um com critério de conclusão)
## Regras           (no positivo; guarda-corpos pareados com o alvo)
## Erros e ambiguidade  (o que fazer, quando perguntar ao usuário)
## Exemplo          (entrada e saída esperada)
```

### 3.7 Invocação e skills roteadoras (WFA)

- **Invocada pelo modelo:** mantém `description`; custo permanente de contexto em troca de ser descoberta sozinha, e outras skills podem alcançá-la.
- **Invocada pela pessoa** (`disable-model-invocation: true`): contexto zero, mas você é o índice e precisa lembrar que ela existe.
- Quando as skills manuais se multiplicam, crie uma **skill roteadora** (também manual) que nomeia as outras e diz quando usar cada uma. Ela só sugere, não dispara.
- Divida uma skill em duas por invocação só se houver um termo-gatilho distinto que você realmente use nos prompts.

**Entregável:** `SKILL.md` enxuto, com a lista do que foi removido e por quê.

## Fase 3b: Decidir o que vira código

A skill é probabilística: o agente interpreta o texto e pode variar. Para cada passo, pergunte:

> *Se eu rodasse este passo 5 vezes, o resultado precisa ser idêntico?* Se sim, é candidato a código.

| Situação | Responsável | Exemplo |
|----------|-------------|---------|
| Várias abordagens válidas; depende de contexto ou julgamento | **Texto na skill** | Revisar um texto, priorizar itens |
| Operação determinística, ou o agente geraria o mesmo código toda vez | **Skill + script em `scripts/`** | Validar arquivo, converter formato, gerar artefato |
| Operação frágil, em que um passo fora de ordem quebra tudo | **Skill + script** (execução exata) | Migração, empacotamento |
| Erro que precisa de mensagem previsível | **Skill + script** | Validador que lista o que falhou e onde |
| Regra **mecânica** que deve valer **mesmo sem a skill ser acionada** | **Fora da skill:** linter, hook ou CI | Proibir um import, exigir nome de arquivo |

- **Sinal vindo dos testes:** se vários testes fazem o agente escrever o mesmo script auxiliar, esse script deve ser empacotado em `scripts/`. (SC)
- O script roda sem carregar seu conteúdo no contexto; só a saída consome tokens. (autor)
- No `SKILL.md`, diga se o agente deve **executar** ou **ler** o script (por padrão, executar) e o que fazer com cada saída. Mensagens de erro devem dizer o que falhou, onde e como corrigir. (autor)
- Teste o script isoladamente antes de colocá-lo na skill. (autor)

## Fase 3c: Um exemplo few-shot

Escolha o melhor caso real e escreva uma seção curta `## Exemplo` com **entrada** e **saída esperada**. Um exemplo bom ensina mais que dez regras abstratas, mas o modelo tende a copiar o formato: use poucos, curtos e, se possível, variados. (SC)

## Fase 4: Validar

Valide em quatro frentes. O `skill-creator` traz ferramentas para as três primeiras.

**1. Validação mecânica (SC).** Rode `scripts/quick_validate.py <pasta-da-skill>`: confere `SKILL.md` e frontmatter, `name` em kebab-case com até 64 caracteres, `description` com até 1024 caracteres e sem `<` ou `>`, e só as chaves permitidas (`name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility`).

**2. Evals com vs. sem skill (SC).**
- Escreva 2 a 3 pedidos realistas em `evals/evals.json`, no tom em que um usuário de verdade escreveria.
- Para cada pedido, rode **uma execução com a skill e uma baseline** (sem skill; ou o snapshot da versão antiga, se você está melhorando uma skill existente), na mesma rodada.
- Defina **asserções objetivamente verificáveis**, com nomes descritivos. Em saídas subjetivas, prefira revisão humana.
- Agregue com `aggregate_benchmark`: taxa de aprovação, tempo e tokens, com média ± desvio. O analisador aponta asserções que passam sempre (não discriminam) e testes instáveis.
- Revise as saídas no visualizador (`eval-viewer/generate_review.py`) e dê feedback por caso. Opcional: **comparação cega** entre duas versões.

**3. Teste de acionamento (SC).** A `description` decide se a skill dispara.
- Crie **20 consultas**: 8 a 10 que **devem** acionar (frases formais e informais, inclusive sem citar a skill) e 8 a 10 que **não** devem. As negativas valiosas são as de **quase-acerto**, que compartilham palavras-chave mas pedem outra coisa.
- Rode `scripts/run_loop` (3 execuções por consulta, divisão 60% treino e 40% teste, até 5 iterações). Ele escolhe a melhor descrição pelo resultado no **teste**, para evitar overfitting.
- Requer o `claude -p`, ou seja, só funciona no Claude Code. Consultas simples demais ("leia este arquivo") não acionam skill nenhuma e são casos de teste fracos.

**4. Leitura qualitativa (autor).** Compare com o baseline em comportamento observável: seguiu os passos? omitiu algo? inventou conteúdo? acionou no momento certo?

| Resultado | Ação |
|-----------|------|
| Skill não foi acionada | Reescrever a `description`; repetir o teste de acionamento |
| Acionada, mas passos ignorados | Tornar o critério de conclusão do passo mais claro; se o passo é exato, mover para script (3b) |
| Resultado varia em um passo que devia ser fixo | Transformar o passo em script |
| Script falha ou o agente não o executa | Corrigir a mensagem de erro e o trecho do `SKILL.md` que manda executá-lo |
| Saída igual ao baseline | A linha é no-op: apague; se nada melhora, descarte a skill |
| Regressão em caso de borda | Restaurar a regra cortada na Fase 3 |

Itere: **corte o que não mudou o resultado, acrescente só o que corrigiu uma falha observada**, e reteste com um baseline que continua sendo o "sem skill". Reteste também ao trocar de modelo, porque o que é no-op depende dele. (autor, com apoio em WFA para a dependência do modelo)

**Entregável:** `testes/` com cenários, resultados por versão e modelo, e a `description` final.

## Fase 5: Empacotar, publicar e manter

```text
nome-da-skill/
├── SKILL.md
├── references/
├── scripts/            # código determinístico chamado pela skill
├── assets/             # arquivos usados na saída (templates, ícones)
└── testes/
    ├── cenarios.md
    └── resultados.md   # com vs. sem skill, por modelo e data
```

- **Empacotar:** `scripts/package_skill.py <pasta-da-skill>` gera o `.skill`. (SC)
- **Atualizar uma skill existente:** preserve o `name` e o nome do diretório; edite uma cópia gravável e use um snapshot da versão antiga como baseline do próximo teste. (SC)
- **Claude:** `name` até 64 caracteres (minúsculas, números e hífens) e `description` até 1.024. (SC, via `quick_validate.py`)
- **Copilot:** `name` e `description` em minúsculas, nome sem a palavra "skill", frontmatter só com esses dois campos, arquivo `agents/ui_metadata.yaml` e pacote `skill.zip` de até 25 MB.
- **Manter (autor):** defina um responsável, registre versão e mudanças, revise periodicamente com a poda da Fase 3.5 e retire a skill quando o modelo passar a fazer sozinho o que ela ensina.

## Segurança

- **Princípio da menor surpresa (SC):** a skill não pode conter malware nem código de exfiltração, e seu conteúdo não deve surpreender o usuário em relação à intenção declarada.
- **Skills rodam com as permissões do agente (autor).** Revise scripts e instruções de qualquer skill de terceiros antes de instalar, nunca inclua segredos e use o menor conjunto de ferramentas que a tarefa exige (`allowed-tools`).
- **Material referenciado pode conter instruções hostis (autor).** Trate documentos externos lidos pela skill como dados, não como ordens.

## Anti-padrões

| Anti-padrão | Sintoma | Correção |
|-------------|---------|----------|
| Sprawl | `SKILL.md` enorme, agente ignora partes | Descer referência para `references/` com ponteiro (WFA) |
| Sedimento | Camadas velhas que ninguém ousa remover | Poda periódica com o teste no-op (WFA) |
| Duplicação | A mesma regra em dois lugares, com prominência inflada | Fonte única da verdade (WFA) |
| Proibição solta | "Não faça X" e o agente faz X | Descrever o comportamento-alvo (WFA) |
| MUST em caixa alta | Regra rígida, frágil fora dos exemplos | Explicar o porquê (SC) |
| Overfitting | Funciona nos seus 3 testes e falha no resto | Generalizar a partir do feedback (SC) |
| Critério de conclusão vago | O agente encerra o passo cedo | Afiar o critério (WFA) |
| Gatilhos repetidos na `description` | Contexto gasto sem acionar mais | Um gatilho por ramo (WFA) |
| Regra mecânica em texto | Só vale quando a skill dispara | Linter, hook ou CI (autor) |
| Script sem mensagem de erro | O agente não consegue se autocorrigir | Dizer o que falhou, onde e como corrigir (autor) |

## Checklist rápido

- [ ] Objetivo, gatilho, saída esperada e não-objetivos escritos (Fase 0)
- [ ] Skill é o artefato certo; invocação (modelo ou pessoa) decidida
- [ ] Baseline sem skill mostra a falha que ela corrige
- [ ] `description` com o quê + quando, um gatilho por ramo, termo-chave no início
- [ ] Corpo enxuto; material de ramo em `references/` com ponteiro dizendo quando ler
- [ ] Cada passo com critério de conclusão claro
- [ ] Instruções no positivo e com o porquê; guarda-corpos pareados com o alvo
- [ ] Poda feita (teste no-op), com a lista do que foi removido
- [ ] Cada passo classificado: texto, script ou checagem fora da skill
- [ ] Scripts testados isoladamente, com erros explícitos
- [ ] 1 exemplo few-shot curto
- [ ] `quick_validate.py` sem erros
- [ ] Evals com vs. sem skill, incluindo caso de borda, e teste de acionamento
- [ ] Segurança revisada (scripts, segredos, ferramentas permitidas)
- [ ] Empacotado conforme a plataforma, com responsável e versão definidos

## Quem cobre o quê

| Tema | `skill-creator` | `writing-for-agents` | Só este guia |
|------|:---:|:---:|:---:|
| Intenção, entrevista, rascunho | ✅ | | |
| Objetivo, gatilho, saída esperada | ✅ | | |
| Não-objetivos, "artefato certo" | | | ✅ |
| Invocação por modelo ou pessoa, skill roteadora | | ✅ | |
| Escrita: imperativo, porquê, evitar caixa alta | ✅ | | |
| Escrita: forma positiva, leading words, fonte única | | ✅ | |
| `description` como ponteiro de contexto | ◐ | ✅ | |
| Divulgação progressiva (3 níveis, 500 linhas) | ✅ | ✅ | |
| Hierarquia, co-localização, sprawl | | ✅ | |
| Critério de conclusão por passo | | ✅ | |
| Poda: no-op, sedimento | ◐ | ✅ | |
| Scripts: quando empacotar | ✅ | | |
| Evals, benchmark, comparação cega | ✅ | | |
| Teste de acionamento (`run_loop`) | ✅ | | |
| Validação de frontmatter | ✅ | | |
| Empacotar `.skill`, atualizar skill existente | ✅ | | |
| Menor surpresa (malware, exfiltração) | ✅ | | |
| Segurança de terceiros, menor privilégio | | | ✅ |
| Manutenção: dono, versão, aposentadoria | | | ✅ |
| Graus de liberdade, convenção de nomes | | | ✅ |

✅ coberto, ◐ parcial.

## Limites deste guia

- As fontes são dois documentos de instrução. Eles dizem **como** os autores recomendam escrever skills, mas não trazem medições de que cada prática melhora o resultado. Por isso a Fase 4 existe: **meça na sua skill.**
- Itens marcados **(autor)** não estão nas duas fontes e não foram verificados contra a documentação oficial linha a linha.
- A otimização da `description` e os benchmarks com baseline dependem do Claude Code e de subagentes.

## Referências

- [`skill-creator` (anthropics/skills)](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
- [`writing-for-agents` (mattpocock/skills)](https://github.com/mattpocock/skills/tree/main/skills/productivity/writing-for-agents) e [artigo no AI Hero](https://www.aihero.dev/skills-writing-for-agents)
- [Skill authoring best practices (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Diagrama gerado com [Archify](https://github.com/tt-a1i/archify): fonte em [`docs/fluxo-skills.workflow.json`](docs/fluxo-skills.workflow.json), versão interativa em [`docs/fluxo-skills.html`](docs/fluxo-skills.html)
