# Processo prático de criação de skills

Pipeline em 5 fases usando três ferramentas, cada uma no papel em que é mais forte:

| Ferramenta | Papel no processo | Quando entra |
|------------|-------------------|--------------|
| `skill-creator` (Anthropic) | **Estruturar:** gera `SKILL.md` com frontmatter válido, gatilhos e casos de teste; também modifica skills existentes e mede desempenho | Fases 2 e 4 |
| `write-a-skill` (Matt Pocock) | **Entrevistar:** levanta requisitos com você, redige o rascunho e revisa em conjunto | Fase 2, quando não há material-fonte |
| `writing-for-agents` (Matt Pocock; antes `writing-great-skills`) | **Lapidar:** aplica o teste *no-op* e corta tudo que o modelo já faz sozinho | Fase 3 |

> **Ideia central:** o gerador estruturado produz um rascunho **completo, porém inchado**; o filtro de poda o transforma em algo **enxuto e previsível**; o teste com vs. sem skill prova que valeu a pena.

## Visão do processo

```mermaid
flowchart TD
    A["Fase 1: Preparar<br/>material-fonte + 3 cenários + baseline SEM skill"] --> B{"Há material-fonte?"}
    B -- "Sim (texto, histórico de chat, docs)" --> C["Fase 2A: skill-creator<br/>extrair regras, fluxo e gatilhos"]
    B -- "Não" --> D["Fase 2B: write-a-skill<br/>entrevista de requisitos + rascunho"]
    C --> E["Fase 3: writing-for-agents<br/>teste no-op, mover volume para references/"]
    D --> E
    E --> F["Fase 3b: 1 exemplo few-shot<br/>entrada e saída esperada"]
    F --> G["Fase 4: teste COM vs. SEM skill<br/>mesmos cenários, sessões isoladas"]
    G --> H{"Melhorou sem regressão?"}
    H -- "Não" --> I["Corrigir: cortar ruído<br/>ou adicionar só o que falhou"]
    I --> G
    H -- "Sim" --> J["Fase 5: empacotar e publicar"]
```

## Fase 1: Preparar (antes de qualquer skill)

1. **Escolha 3 cenários reais** em que a IA erra, varia ou esquece algo. Inclua 1 caso de borda.
2. **Rode cada cenário sem skill**, em sessão nova, e guarde as saídas. É o seu baseline.
3. **Reúna o material-fonte**, se existir (Markdown de documentação, histórico de uma tarefa bem-sucedida, regras internas).
4. **Decida:** se o baseline já sai bom, pare aqui; a skill não se justifica.

**Entregável:** pasta com `cenarios.md` (3 pedidos) e `baseline/` (saídas sem skill).

## Fase 2: Estruturar

### 2A. Com material-fonte → `skill-creator`

Prompt sugerido:

```text
Use a skill skill-creator. Material-fonte em anexo.
Extraia apenas: (1) regras de decisão que a IA não aplica bem sozinha,
(2) o fluxo passo a passo, (3) frases de gatilho para a description.
Ignore explicações, histórias e contexto. Coloque o que for volumoso em references/.
Gere o SKILL.md com frontmatter e proponha 3 casos de teste.
```

### 2B. Sem material-fonte → `write-a-skill`

Prompt sugerido:

```text
Use a skill write-a-skill. Quero uma skill para [tarefa].
Faça perguntas até ter: tarefa/domínio, casos de uso, o que precisa de script
e que material de referência existe. Depois redija o rascunho e revise comigo.
```

**Entregável:** rascunho de `SKILL.md` (esperado: longo e redundante; é normal) + pasta `references/` se houver.

## Fase 3: Lapidar com `writing-for-agents`

Prompt sugerido:

```text
Use a skill writing-for-agents. Revise o SKILL.md em anexo.
Aplique o teste no-op linha por linha: se remover a linha não muda o
comportamento do agente, apague. Corte conselhos genéricos, introduções,
conclusões e regras repetidas com outras palavras.
Mova conteúdo útil só em um ramo do fluxo para references/, deixando no
SKILL.md um ponteiro dizendo quando ler. Liste o que removeu e por quê.
```

Regras que a revisão deve garantir:

| Item | Critério |
|------|----------|
| `description` | Terceira pessoa; diz o que faz **e** quando usar; todos os gatilhos ficam aqui |
| Corpo | Menos de 500 linhas; só fluxo, regras não óbvias e casos de borda |
| `references/` | Um nível de profundidade; sumário se passar de 100 linhas |
| `scripts/` | Para operações frágeis ou repetitivas (só a saída consome contexto) |

### Fase 3b: Um exemplo few-shot

Escolha o melhor caso real do material e escreva uma seção curta `## Exemplo` com **entrada** e **saída esperada**. Um exemplo bom ensina mais que dez regras abstratas, mas o modelo tende a copiar o formato: use poucos, curtos e, se possível, variados.

## Fase 4: Validar com vs. sem skill

1. Instale a skill e rode os **mesmos 3 cenários** da Fase 1, em sessões novas e com o mesmo modelo.
2. Compare com o baseline em **comportamento observável**: seguiu os passos? omitiu algo? inventou conteúdo? acionou a skill no momento certo?
3. Opcional: peça ao `skill-creator` para medir o desempenho dos casos de teste.

| Resultado | Ação |
|-----------|------|
| Skill não foi acionada | Reescrever a `description` (gatilhos) |
| Acionada, mas passos ignorados | Tornar o passo mais explícito ou mover para script |
| Saída igual ao baseline | A linha é no-op: apague; se nada melhora, descarte a skill |
| Regressão em caso de borda | Restaurar a regra cortada na Fase 3 |

Itere: **cortar o que não mudou o resultado, acrescentar só o que corrigiu uma falha observada**, e reteste. Reteste também ao trocar de modelo, porque o que é no-op depende dele.

## Fase 5: Empacotar e publicar

Estrutura do repositório:

```text
nome-da-skill/
├── SKILL.md
├── references/
├── scripts/
└── testes/
    ├── cenarios.md
    └── resultados.md   # com vs. sem skill, por modelo e data
```

- Registre em `resultados.md` o baseline, o resultado com skill e o modelo usado.
- Para o **Copilot**: `name` e `description` em minúsculas, nome sem a palavra "skill", frontmatter apenas com esses dois campos, arquivo `agents/ui_metadata.yaml` e pacote `skill.zip` de até 25 MB.
- Para o **Claude**: `name` até 64 caracteres (minúsculas, números e hífens) e `description` até 1.024 caracteres.

## Checklist rápido

- [ ] Baseline sem skill mostra a falha que ela corrige
- [ ] Rascunho gerado por `skill-creator` ou `write-a-skill`
- [ ] Poda feita com `writing-for-agents`, com lista do que foi removido
- [ ] Gatilhos todos na `description`
- [ ] 1 exemplo few-shot curto
- [ ] Teste com vs. sem skill registrado, incluindo caso de borda
- [ ] Empacotado conforme a plataforma de destino

## Referências

- [Skill authoring best practices (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [/writing-for-agents (AI Hero)](https://www.aihero.dev/skills-writing-for-agents)
- [mattpocock/skills](https://github.com/mattpocock/skills)
