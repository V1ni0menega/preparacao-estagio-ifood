# Aula 06 — Tratamento de Erros, Retries Nativos e Deduplicação

**Trilha:** Orquestração de IA  
**Ferramentas:** n8n (`Settings → Retry on Fail`, `Remove Duplicates`, `If`), Gemini API  
**Status:** Concluída (em 11/11/2026).

---

## 1. Objetivo da aula

Fechar a **Etapa 3 (Orquestração de IA)** blindando o pipeline do **TechPulse** contra os dois problemas mais comuns de sistemas em produção:
1. **Falhas transitórias e instabilidade de rede:** configurar tentativas automáticas (*Retries com espera*) dentro do próprio node `HTTP Request` antes de acionar o fallback;
2. **Notícias repetidas:** implementar o node **Remove Duplicates** para garantir que uma notícia já processada não seja enviada novamente para o Gemini, economizando cota e evitando boletins duplicados.

---

## 2. Conceitos

### As 3 Camadas de Defesa em Pipelines de IA

Em sistemas profissionais de GenAI, uma chamada de modelo nunca deve falhar silenciosamente nem derrubar a aplicação. Trabalhamos com **três camadas complementares de defesa**:

```text
[Requisição ao Gemini]
       ↓ (Falha transitória / 503 temporário)
[Camada 1: Retry Automático]  → Tenta de novo 2 ou 3 vezes com intervalo de tempo
       ↓ (Persistiu o erro / Cota estourada 429)
[Camada 2: Fallback de Modelo] → Desvia para modelo reserva (ex.: Flash Lite)
       ↓ (Ambos falharam / Erro crítico)
[Camada 3: Rota de Erro Graciosa] → Captura no If e gera registro de erro sem travar o fluxo
```

Você já construiu a **Camada 2** e a **Camada 3** nas Aulas 03 e 04! Hoje vamos ativar a **Camada 1** no n8n.

---

### O que é *Retry on Fail* e *Exponential Backoff*?

- **Retry on Fail (Tentativa na Falha):** Quando uma API retorna erro momentâneo (ex.: `503 Service Unavailable` por pico de tráfego temporário), repetir a mesma requisição 1 ou 2 segundos depois tem alta chance de sucesso.
- **Intervalo entre tentativas:** Em vez de disparar novas requisições instantâneas (o que pioraria a sobrecarga do servidor), o n8n permite configurar uma pausa (ex.: 2000ms a 5000ms) entre as tentativas.

---

### Deduplicação: Por que é obrigatória antes da IA?

Um feed RSS publica as últimas notícias. Se o TechPulse rodar de hora em hora (via *Schedule Trigger*):
- Às 08:00, ele lê as notícias A, B e C.
- Às 09:00, o feed ainda terá A, B e C (mais a notícia D nova).

Sem deduplicação:
- Você gastaria cota e dinheiro do Gemini reclassificando A, B e C toda hora;
- Seu boletim final acumularia notícias repetidas para o usuário.

**Regra de Engenharia:** Deduplique dados na fonte usando um identificador único imutável (no caso de notícias, a URL única do campo `link`).

---

## 3. Exercícios práticos

### Exercício 1 — Configurar *Retry on Fail* no `HTTP Request`

Vamos ensinar o n8n a tentar novamente antes de desistir e mandar o erro para o fallback:

1. Abra o node **`HTTP Request`** (principal do Gemini).
2. Vá até a aba **Settings** (no topo do painel do node).
3. Ative as seguintes opções:
   - **Retry On Fail:** ligue a chavinha (verde).
   - **Max Tries:** `3` (ele tentará até 3 vezes antes de emitir erro).
   - **Wait Between Tries (ms):** `2000` (aguarda 2 segundos entre cada tentativa).
4. Mantenha **On Error** configurado como `Continue (using error output)` (para que, se todas as 3 tentativas falharem, o fluxo ainda caia suavemente no fallback `HTTP Request1`).
5. *(Opcional)* Faça o mesmo no **`HTTP Request1`** (fallback) para dar a ele a mesma resiliência.

---

### Exercício 2 — Adicionar o node `Remove Duplicates` (Deduplicação de Notícias)

Vamos garantir que o pipeline só processe notícias inéditas:

1. No canvas do n8n, localize a ligação entre o **`RSS Read`** e o node **`Limit`**.
2. Clique no sinal de **`+`** nessa linha e adicione o node **Remove Duplicates**.
3. Na configuração do **Remove Duplicates**:
   - **Mode:** `Keep Unique`
   - **Compare:** selecione `Selected Fields`
   - **Fields to Compare:** adicione o campo:
     ```text
     link
     ```
   *(A URL da notícia é o identificador mais confiável para deduplicação, pois títulos podem ter pequenas variações de edição).*
4. Em **Scope**, mantenha o padrão para limpar duplicatas da execução atual.

---

### Exercício 3 — Validação Final do Pipeline Completo

Agora teste a execução integrada de ponta a ponta:

1. Clique em **Execute workflow** no botão principal inferior.
2. Observe o fluxo fluindo:
   - `RSS Read` coleta os artigos;
   - `Remove Duplicates` elimina qualquer link repetido;
   - `Limit` corta o lote no tamanho seguro (ex.: 3 itens);
   - `HTTP Request` processa com retries embutidos;
   - Se houver falha, `HTTP Request1` assume com Flash Lite;
   - `If` valida se há resposta real da IA;
   - `Edit Fields` faz o parse limpo do JSON;
   - `If1` separa `relevante == true`;
   - `Aggregate` consolida os aprovados no Boletim TechPulse final!

---

## 4. O que deve ficar no diário (Conceitos Técnicos)

- **As 3 Camadas de Defesa:** Retry automático (falhas transitórias) → Fallback de modelo (falhas persistentes/cota) → Degradação graciosa (rota de erro).
- **Retry on Fail com Backoff:** Por que esperar alguns segundos antes de tentar de novo evita agravar sobrecargas em APIs externas.
- **Deduplicação como Otimização de Custo:** Por que deduplicar na entrada (por `link`) protege o orçamento de tokens e a experiência do usuário final.
