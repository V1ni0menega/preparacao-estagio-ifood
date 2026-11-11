# Aula 07 — Conceitos Fundamentais de Agentes de IA vs. Workflows Tradicionais

**Trilha:** Agentes de IA e Function Calling  
**Status:** Em andamento (11/11/2026).

---

## 1. Objetivo da aula

Compreender o salto conceitual da automação: sair dos **Workflows Tradicionais** (onde o caminho é 100% desenhado pelo programador) e entrar nos **Agentes de IA** (onde o modelo recebe um objetivo e decide de forma autônoma quais passos e ferramentas utilizar para alcançá-lo).

---

## 2. O Salto de Paradigma: Workflow vs. Agente

Até a Aula 06, você construiu um **Workflow Tradicional com IA**:
- Você decidiu que o `RSS Read` conecta no `Limit`;
- Você decidiu que o `Limit` conecta no `HTTP Request`;
- Você decidiu a condição do `If1`.

Se uma notícia precisar de uma busca no Google para tirar uma dúvida antes de classificar, o workflow não consegue fazer isso sozinho a menos que **você** adicione manualmente um node de busca.

| Característica | Workflow Tradicional (com ou sem IA) | Agente de IA (AI Agent) |
|---|---|---|
| **Quem define a rota?** | O desenvolvedor (estática/pré-definida). | O próprio modelo de IA (dinâmica/autônoma). |
| **Ponto de partida** | *"Execute o passo A, depois B, se der erro vá para C"*. | *"Aqui está o seu objetivo e as ferramentas que você pode usar. Descubra o caminho"*. |
| **Previsibilidade** | Muito alta (100% determinístico no caminho). | Flexível (adapta-se a cenários imprevistos). |
| **Custo e Latência** | Baixo e previsível (número fixo de nós). | Mais alto e variável (pode precisar de múltiplos ciclos de pensamento). |
| **Melhor uso** | Processos repetitivos com regras bem definidas. | Tarefas complexas, investigativas e com caminhos abertos. |

---

## 3. O Ciclo ReAct (Reasoning + Acting)

Como um agente "pensa" e age no mundo real? A arquitetura mais famosa e utilizada na indústria chama-se **ReAct** (*Reasoning + Acting*):

```text
               ┌──────────────────────────────┐
               │    OBJETIVO DO USUÁRIO       │
               └──────────────┬───────────────┘
                              │
                    ┌─────────▼─────────┐
             ┌─────►│ Thought (Pensar)  │
             │      │ "O que devo fazer │
             │      │  para avançar?"   │
             │      └─────────┬─────────┘
             │                │
┌────────────┴──────────┐     │
│ Observation (Observar)│◄────┤ Decide chamar uma ferramenta
│ "Qual foi o resultado │     ▼
│  da ferramenta?"      │┌─────────────────┐
└───────────────────────┘│  Action (Agir)  │ → Executa uma Tool
                         │ Executa a Tool  │   (API, busca, código)
                         └─────────────────┘
                              │
               (Quando atinge a resposta final)
                              ▼
                         [ RESPOSTA ]
```

1. **Thought (Pensamento):** O modelo avalia a situação atual (*"O usuário pediu as últimas notícias do iFood, mas meu conhecimento é até 2025. Preciso buscar na web"*).
2. **Action (Ação):** O modelo escolhe uma ferramenta disponível e passa os argumentos (*"Chamar ferramenta `buscar_web(query='iFood últimas notícias 2026')`"*).
3. **Observation (Observação):** O sistema executa a ferramenta e devolve o resultado cru para a janela de contexto do modelo.
4. **Loop:** O modelo lê a observação e pensa novamente. Se a informação foi suficiente, devolve a resposta final. Se não, toma outra ação!

---

## 4. A Anatomia de um Agente de IA

Um Agente de IA completo é composto por **4 pilares fundamentais**:

```text
               ┌─────────────────────────────────┐
               │       1. CÉREBRO (LLM)          │
               │  Raciocínio, tomada de decisão  │
               │       e compreensão natural     │
               └───────────────┬─────────────────┘
                               │
       ┌───────────────────────┼───────────────────────┐
       ▼                       ▼                       ▼
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│ 2. FERRAMENTAS│       │  3. MEMÓRIA  │       │4. PLANEJAMENTO│
│    (Tools)   │       │   (Memory)   │       │  (Planning)  │
│ APIs, bancos,│       │ Curto prazo  │       │ Decomposição │
│ busca web,   │       │ (contexto) e │       │ de problemas │
│ calculadoras │       │ Longo prazo  │       │ e auto-crítica│
└──────────────┘       └──────────────┘       └──────────────┘
```

1. **Cérebro (Core Model / LLM):** É o motor de inteligência (ex.: Gemini, Claude) responsável por interpretar, planejar e decidir.
2. **Ferramentas (Tools / Function Calling):** A forma como o agente interage com o mundo externo (APIs, envio de e-mail, consulta SQL, busca na web). O modelo **não executa** a API diretamente; ele gera uma instrução estruturada pedindo ao sistema para executá-la.
3. **Memória (Memory):**
   - *Curto Prazo:* O histórico imediato da conversa e os passos anteriores do loop ReAct.
   - *Longo Prazo:* Bancos vetoriais (RAG) ou históricos persistidos para lembrar preferências de sessões anteriores.
4. **Planejamento (Planning & Self-Correction):** A capacidade do modelo de quebrar uma meta grande em sub-tarefas e corrigir o curso quando uma ferramenta retorna erro.

---

## 5. Pergunta de Ouro em Entrevistas: Quando usar Workflow vs. Quando usar Agente?

> 💡 **Dica de Entrevista no iFood:** Empresas maduras de tecnologia não querem quem tenta colocar "Agentes autônomos" em tudo. Elas querem quem saiba escolher a ferramenta certa para o problema certo.

- **Use Workflows quando:**
  - O processo tem início, meio e fim conhecidos (ex.: pipeline do TechPulse de hoje: ler RSS → filtrar → resumir → salvar);
  - Latência baixa e previsibilidade de custos forem prioridade máxima;
  - Não houver tolerância a "alucinações de decisão" (processos financeiros, emissão de notas, cadastros rígidos).

- **Use Agentes quando:**
  - O caminho exato para a solução não pode ser previsto no código (ex.: suporte ao cliente complexo, investigação de fraudes, assistente que pesquisa e cruza fontes diferentes);
  - A tarefa exige tentativa e erro orientada por dados do ambiente;
  - O usuário interage em linguagem natural aberta e pede ações variadas.

---

## 6. Próximo passo (Ponte para a Aula 08)

Na Aula 08 (amanhã, 12/11), veremos a peça técnica que torna os Agentes possíveis na prática: **Function Calling (Chamada de Ferramentas)**. Vamos entender como o Gemini avisa ao código que quer disparar uma função externa com parâmetros estruturados.

