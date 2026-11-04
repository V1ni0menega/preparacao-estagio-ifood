# Aula 03 — Integração com RSS e primeiros passos com IA (Gemini)

**Trilha:** Orquestração de IA
**Ferramentas:** n8n, RSS, Gemini API
**Status:** A fazer.

## 1. Objetivo da aula

Dar o primeiro passo do TechPulse de verdade: ler notícias de uma fonte RSS real e enviar o conteúdo para um modelo de IA (Gemini) processar — unindo tudo que já foi praticado (HTTP Request, Webhook, Edit Fields, credenciais) num pipeline só.

## 2. Conceitos

### RSS

RSS é um formato (XML) que sites de notícias publicam automaticamente, listando os últimos conteúdos (título, link, data, resumo). É a fonte de dados "de fundo" do TechPulse.

No n8n existem dois nodes relacionados (já vistos na Aula 01):
- **RSS Feed Read** — consulta uma fonte RSS sob demanda (bom para Schedule Trigger, ex.: rodar toda manhã).
- **RSS Trigger** — inicia o workflow quando detecta uma publicação nova.

Hoje vamos usar o **RSS Feed Read**, puxando manualmente pra entender o formato do dado antes de automatizar o agendamento.

### De API para IA: o que muda

Até agora você consumiu APIs que devolvem dado estruturado e previsível (`agify.io`, por exemplo, sempre devolve `{name, age, count}`).

Uma API de modelo de IA (como o Gemini) é diferente: você manda um **prompt** (texto em linguagem natural) e recebe de volta uma **resposta gerada**, que pode variar a cada chamada. O "contrato" da API ainda é JSON (request e response), mas o conteúdo dentro é texto livre — por isso track de prompt engineering e validação de resposta importam tanto nessa trilha.

### Credenciais (aplicando o que já foi alertado na Aula 01)

A API key do Gemini **nunca** deve ir direto na URL ou no corpo do node. Vamos guardá-la como credencial no n8n.

## 3. Preparação (antes dos exercícios)

1. Gere uma API key gratuita em [Google AI Studio](https://aistudio.google.com/apikey).
2. No n8n: **Settings → Credentials → New**.
3. Procure por **"Header Auth"** (autenticação genérica via header HTTP).
4. Configure:
   - Name: `x-goog-api-key`
   - Value: *(sua API key)*
5. Salve a credencial com um nome tipo "Gemini API Key".

Isso cria uma camada de autenticação reutilizável, sem expor a chave em nenhum node.

## 4. Exercícios práticos

### Exercício 1 — Ler uma fonte RSS real

1. Novo workflow. Adicione um **Manual Trigger**.
2. Adicione o node **RSS Feed Read**.
3. Use uma URL de feed real, por exemplo:
   - `https://www.tecmundo.com.br/rss`
   - ou `https://feeds.feedburner.com/TechCrunch/` (TechCrunch)
4. Execute e observe a estrutura do retorno: cada item costuma ter `title`, `link`, `pubDate`, `contentSnippet`.

**Pergunta pra responder no diário:** quantos itens vieram? Qual campo você usaria como "resumo" se não tivesse IA nenhuma?

### Exercício 2 — Enviar uma notícia pro Gemini

1. Adicione um node **HTTP Request** depois do RSS Feed Read.
2. Method: `POST`
3. URL: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
   (confira o nome do modelo mais atual em [ai.google.dev/gemini-api/docs/models](https://ai.google.dev/gemini-api/docs/models) — pode ter mudado)
4. Authentication: **Generic Credential Type** → **Header Auth** → selecione a credencial "Gemini API Key" criada antes.
5. Send Body: ligado, modo JSON:
```json
{
  "contents": [
    {
      "parts": [
        { "text": "Resuma esta notícia em até 2 frases, em português: {{ $json.title }} — {{ $json.contentSnippet }}" }
      ]
    }
  ]
}
```
6. Execute e veja a resposta. O texto gerado vem em um caminho parecido com:
```
candidates[0].content.parts[0].text
```

### Exercício 3 — Organizar o resultado

Adicione um **Edit Fields** depois do Gemini, combinando:
- `titulo_original`: `{{ $('RSS Feed Read').item.json.title }}`
- `link`: `{{ $('RSS Feed Read').item.json.link }}`
- `resumo_ia`: `{{ $json.candidates[0].content.parts[0].text }}`

Esse é o formato final de um item do TechPulse: notícia original + resumo gerado por IA, pronto pra ser armazenado.

## 5. Observação sobre o fluxo completo

Hoje você processou **um item por vez** manualmente (RSS Feed Read retorna uma lista, mas só olhamos o primeiro). O n8n processa listas automaticamente em cada node (cada item passa pela mesma cadeia) — então, ao rodar numa fonte com várias notícias, o mesmo workflow já tentaria chamar o Gemini para cada uma. Vale observar esse comportamento, mas sem se preocupar ainda com controle de custo/rate limit — isso entra na Etapa 3 mais adiante (tratamento de erros, 10/11).

## 6. O que deve ficar no diário

- Estrutura de um item retornado pelo RSS Feed Read.
- Diferença entre consumir uma API "de dado" e uma API "de IA" (prompt → resposta variável).
- Caminho da resposta do Gemini (`candidates[0].content.parts[0].text`) e por que isso é mais frágil que `age` ou `name` de antes (depende da estrutura que o provedor define e pode mudar).
- Dificuldades com a credencial de Header Auth, se houver.
