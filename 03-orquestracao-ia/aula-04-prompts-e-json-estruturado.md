# Aula 04 — Engenharia de Prompts e Respostas Estruturadas em JSON

**Trilha:** Orquestração de IA
**Ferramentas:** n8n, RSS, Gemini API (`generationConfig` + JSON)
**Status:** Concluída (em 08/11/2026).

## 1. Objetivo da aula

Evoluir a integração com o Gemini construída na Aula 03: sair de uma resposta em **texto livre** (que só serve para leitura humana) e fazer o modelo devolver um **JSON estruturado e previsível** (com `categoria`, `relevante`, `resumo` e `tags`), permitindo que os próximos nodes do workflow tomem decisões automáticas com base na análise da IA.

---

## 2. Aquecimento: Limpeza rápida da Aula 03 (2 min)

Antes de mexer no prompt, abra o workflow da Aula 03 em `http://localhost:5678` e feche as duas pendências deixadas no final da última sessão:

1. **No node `HTTP Request1` (fallback):** troque o nome do modelo na URL (que estava como `gemini-xyz-invalido` para testar o erro 404) de volta para `gemini-flash-lite-latest`:
   ```text
   https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-lite-latest:generateContent
   ```
2. **No node `Edit Fields` final (branch False / erro):** corrija o nome do campo de `"status: "` (com dois-pontos e espaço) para apenas `"status"`.

---

## 3. Conceitos

### Por que texto livre é um problema em automações?

Na Aula 03, pedimos ao Gemini: *"Resuma esta notícia em até 2 frases"*.
Se hoje pedirmos em texto livre: *"Classifique a categoria, diga se é relevante e resuma"*, o modelo pode responder de várias formas diferentes:

- Tentativa 1: `"Categoria: IA\nRelevante: Sim\nResumo: A notícia fala sobre..."`
- Tentativa 2: `"Claro! Aqui está a análise:\n- **Categoria:** Inteligência Artificial..."`

Para um humano lendo no chat, as duas respostas são ótimas. Mas para um sistema automatizado (como o **TechPulse**), isso é um pesadelo: como um node `If` ou um banco de dados vai saber onde começa a categoria e onde termina o resumo se o formato muda a cada execução?

**Solução:** exigir que o modelo responda estritamente em **JSON**, seguindo um contrato (schema) definido por nós.

---

### Anatomia de um bom prompt para automação

Em automação, um prompt não é uma "conversa" — é uma **especificação técnica**. Dividimos o prompt em 4 blocos claros:

1. **Papel (Role):** Quem o modelo é (*"Você é um curador técnico de notícias do sistema TechPulse"*).
2. **Tarefa (Task):** O que ele deve analisar (*"Analise o título da notícia abaixo"*).
3. **Critérios / Regras de negócio (Rules):**
   - Categorias permitidas (lista fechada, ex.: `["IA", "Engenharia de Software", "Cloud & DevOps", "Dados", "Segurança", "Outros"]`).
   - O que define `relevante: true` vs. `relevante: false` (ex.: notícias técnicas são relevantes; promoções de varejo, fofocas ou lançamentos de celulares comuns não são).
4. **Formato de saída (Output Contract):** Exemplo explícito das chaves esperadas no JSON.

---

### Duas camadas de garantia de JSON no Gemini

Se você pedir JSON **apenas no texto do prompt**, o modelo frequentemente devolve o JSON cercado por blocos de Markdown:

````text
```json
{ "categoria": "IA", ... }
```
````

Esses acentos graves (`` ```json ``) fazem qualquer `JSON.parse()` quebrar!

Para garantir que a resposta seja **100% JSON puro**, combinamos duas camadas na chamada da API do Gemini:

1. **No Prompt:** descrevemos os campos e regras de negócio.
2. **No `generationConfig` da API:** enviamos o parâmetro `"responseMimeType": "application/json"`. Isso trava o motor do Gemini para que ele só consiga gerar um JSON sintaticamente válido, sem texto introdutório e sem blocos Markdown.

---

### O detalhe do "JSON dentro de JSON" (`JSON.parse`)

Quando usamos `responseMimeType: "application/json"`, a estrutura do envelope da API do Gemini não muda — o campo `candidates[0].content.parts[0].text` continua sendo do tipo **string**, só que agora o texto dentro dessa string é um objeto JSON serializado:

```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "{\"categoria\":\"IA\",\"relevante\":true,\"resumo\":\"...\"}"
          }
        ]
      }
    }
  ]
}
```

Por isso, dentro do n8n, para acessar `categoria` ou `relevante` individualmente, usamos a função nativa do JavaScript **`JSON.parse()`** dentro da expressão:

```javascript
{{ JSON.parse($json.candidates[0].content.parts[0].text).categoria }}
```

---

## 4. Exercícios práticos

### Exercício 1 — Prompt estruturado + `responseMimeType`

1. No seu workflow da Aula 03, abra o node **HTTP Request** (principal) e depois faça o mesmo no **HTTP Request1** (fallback).
2. No campo **JSON Body**, substitua o payload antigo pelo payload abaixo, que inclui instruções claras de classificação e o bloco `generationConfig`:

```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "Você é um curador técnico de notícias de tecnologia do projeto TechPulse. Analise a notícia a seguir e retorne um JSON com exatamente estes campos:\n- \"categoria\": escolha estritamente uma entre [\"IA\", \"Engenharia de Software\", \"Cloud e Infra\", \"Segurança\", \"Negócios Tech\", \"Outros\"]\n- \"relevante\": booleano (true se for relevante para desenvolvedores ou profissionais de tecnologia/IA; false se for promoção de produto, entretenimento ou assunto genérico)\n- \"resumo\": string com resumo claro e objetivo em português de até 2 frases\n- \"tags\": array com até 3 palavras-chave curtas em português\n\nNotícia: {{ $('RSS Read').item.json.title }}"
        }
      ]
    }
  ],
  "generationConfig": {
    "responseMimeType": "application/json",
    "temperature": 0.2
  }
}
```

> **Dica importante (`$('RSS Read').item.json.title`):** referenciar direto o node `'RSS Read'` garante que tanto o `HTTP Request` principal quanto o `HTTP Request1` (fallback) sempre peguem o título original da notícia, mesmo quando o primeiro node falha.
>
> **Por que `temperature: 0.2`?** Temperatura baixa (próxima de 0) deixa o modelo mais determinístico e consistente para tarefas de classificação e extração de dados.

3. Clique em **Execute step** e observe o `Output`: veja que `candidates[0].content.parts[0].text` agora contém um JSON limpo, sem `` ```json `` ao redor.

---

### Exercício 2 — Desempacotar os campos no `Edit Fields`

Agora vamos transformar essa string JSON em colunas reais do nosso item no n8n.

1. Abra o node **Edit Fields** que fica na saída **True** do node `If`.
2. Configure os campos (Manual Mapping) usando `JSON.parse`:

- **`titulo_original`** (String):
  `{{ $('RSS Read').item.json.title }}`
- **`link`** (String):
  `{{ $('RSS Read').item.json.link }}`
- **`categoria`** (String):
  `{{ JSON.parse($json.candidates[0].content.parts[0].text).categoria }}`
- **`relevante`** (Boolean — ou String, se preferir visualizar rápido):
  `{{ JSON.parse($json.candidates[0].content.parts[0].text).relevante }}`
- **`resumo_ia`** (String):
  `{{ JSON.parse($json.candidates[0].content.parts[0].text).resumo }}`
- **`tags`** (Array / String):
  `{{ JSON.parse($json.candidates[0].content.parts[0].text).tags }}`

3. Execute o node e confira o resultado na aba **Table** ou **JSON**: todos os atributos gerados pela IA agora estão lado a lado com o `titulo_original` e o `link` do RSS!

---

### Exercício 3 — Desafio: Filtrar notícias com base na decisão da IA

Até aqui a IA apenas "enriqueceu" o dado. Agora ela vai **controlar o fluxo do workflow**:

1. Adicione um novo node **If** (ou **Filter**) *depois* do `Edit Fields` de sucesso.
2. Configure a condição para verificar o que a IA decidiu:
   - Opção A (por booleano): `{{ $json.relevante }}` **is true** (atenção ao tipo Boolean no dropdown do If!).
   - Opção B (por texto/categoria): `{{ $json.categoria }}` **is not equal to** `Outros`.
3. Teste o workflow!

**Por que isso é um marco importante?**
Em uma automação tradicional (Aula 01/02), o `If` só conseguia testar regras rígidas como `idade > 19`. Agora, o `If` toma decisões baseadas em **interpretação semântica de texto** feita pelo Gemini, mas com a segurança de um campo tipado (`relevante` / `categoria`).

---

## 5. O que deve ficar no diário

- Por que respostas em texto livre são frágeis para automações e como o contrato JSON resolve isso.
- O papel do `responseMimeType: "application/json"` e da `temperature` baixa no `generationConfig`.
- Como blindar aspas dentro do prompt com `.replace(/"/g, "'")`.
- Como usar `JSON.parse($json.candidates[0].content.parts[0].text)` no n8n para extrair cada propriedade gerada pelo modelo.
- Exemplo real do JSON final produzido pelo seu workflow no Exercício 2/3.

