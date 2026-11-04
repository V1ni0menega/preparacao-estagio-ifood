# Diário de Aprendizado

## 30/10/2026 — Etapa 1, Aula 01: Fundamentos de automação com n8n

> Conteúdo completo da aula e exercícios de modelagem em [01-automacao/aula-01-fundamentos.md](../01-automacao/aula-01-fundamentos.md).

### Automação e Orquestração de Sistemas

1. **Automação** — executar tarefas/processos repetitivos com pouca ou nenhuma intervenção humana, ganhando consistência e reduzindo erro manual.
2. **Workflow** — o processo inteiro, representado como uma sequência de etapas (nodes) conectadas.
3. **Node** — uma etapa individual do workflow, responsável por uma operação específica (ex.: HTTP Request, IF, Filter).
4. **Trigger** — o que decide *quando* o workflow começa (Schedule, Webhook, Manual, RSS Trigger).
5. **Action** — o que decide *o que fazer* dentro do workflow (consultar API, salvar dado, enviar e-mail).
6. **JSON** — formato de dados que circula entre os nodes; cada node pode consumir o output do anterior (ex.: `{{ $json.title }}`).
7. **API e HTTP Request** — forma de integrar o workflow com sistemas externos, usando métodos GET/POST/PUT/PATCH/DELETE.
8. **Condicionais** — o node IF, que avalia uma condição e direciona o fluxo por caminhos diferentes (True/False).
9. **Credenciais** — forma seguro de autenticar em serviços externos (API keys, tokens, OAuth) sem expor segredos no código.
10. **Tratamento de erros** — prever falhas (API fora do ar, token inválido, limite de requisição) e decidir a reação (retry, log, notificação).

### Exercício 1 — Questionário de fixação

Ainda não respondido formalmente — os conceitos foram absorvidos e aplicados direto nos exercícios de modelagem abaixo.

### Exercício 2 — Workflow do TechPulse

1. **Schedule Trigger** — inicia a execução diariamente às 8h.
2. **RSS Feed Read** — consulta a fonte RSS de notícias.
3. **Filter** — seleciona apenas notícias relevantes (tecnologia).
4. **Gemini** — classifica a notícia e gera um resumo.
5. **Database** — armazena o resultado.

### Exercício 3 — Desafio: automação de atendimento/cadastro

1. **Form Trigger / Webhook** — recebe os dados do formulário.
2. **IF** — valida se o e-mail foi preenchido e é válido.
   - **True:** armazena o cliente, define status, envia confirmação por e-mail.
   - **False:** registra como pendente, solicita e-mail válido.

### Dificuldades encontradas

Nenhuma barreira conceitual grave; o ponto de atenção foi diferenciar **RSS Feed Read** (consulta agendada) de **RSS Trigger** (reage a nova publicação) — ambos parecem a mesma coisa à primeira vista, mas têm gatilhos diferentes.

### Aprendizados e próximos passos

Entendido o vocabulário base (workflow, node, trigger, action, JSON, API, IF, credenciais, erros) e a diferença entre automação tradicional e automação com IA embarcada (classificação/resumo).

**Próximo passo:** sair da modelagem no papel e abrir o n8n de fato — instalar (self-hosted local via Docker, conforme recomendado na aula) e construir o primeiro workflow real com Trigger + HTTP Request + IF.

**Atenção ao cronograma:** este conteúdo era do dia 30/10, mas foi concluído em 01/11 — ajustar o ritmo ou compactar as próximas etapas para não comprometer o prazo de inscrição (17/11).

## 01/11/2026 — Primeiro workflow real no n8n

Construído e testado no n8n (self-hosted, Node 26 LTS): **Manual Trigger → Edit Fields (nome, email, idade) → If (idade > 19) → Edit Fields (status: aprovado | status: pendente)**.

Validação:
- `idade = 19` → caiu no **False Branch** → `status: pendente` ✓
- `idade = 20` → caiu no **True Branch** → `status: aprovado` ✓

Workflow executado com sucesso nos dois cenários, confirmando o entendimento de condicionais (IF) e branches na prática. Primeira entrega da Etapa 1 cumprida.

**Próximo passo:** trocar o Edit Fields inicial por um node HTTP Request consumindo uma API real.

## 01/11/2026 (cont.) — HTTP Request com API real

Workflow evoluído para: **Manual Trigger → HTTP Request (GET api.agify.io?name=vinicius) → Edit Fields → If (age > 19) → Edit Fields (status: aprovado | pendente)**.

Erro encontrado: após trocar o HTTP Request, o fluxo continuou caindo no False Branch mesmo com `age: 29` retornado pela API — o Edit Fields do meio ainda tinha o campo `idade` fixado manualmente em 19 (resquício do exercício anterior) e o If ainda lia `{{ $json.idade }}` em vez de `{{ $json.age }}`. Corrigido removendo o valor manual e trocando a expressão do If para o campo correto.

Validação final: `age: 29` → **True Branch** → `status: aprovado` ✓. Workflow executado com sucesso.

**Aprendizado:** dado estático deixado em um node anterior pode mascarar silenciosamente um dado novo vindo de uma API — sempre conferir se o nome do campo usado nas expressões bate com o que a fonte de dados realmente retorna.

**Próximo passo:** Webhook (etapa 2) e explorar mais a fundo tratamento de erros/condições na resposta da API (ex.: o que fazer se a API não retornar `age`).

## 01/11/2026 (cont.) — Interface do n8n

Revisão guiada da interface: barra lateral (Assistant, Overview, Templates, Insights, Settings → Credentials), barra superior do workflow (Editor, Executions, Evaluations, Publish) e painel de detalhe do node (Input/Output, Schema/Table/JSON, Execute step, Docs).

Guia de referência completo salvo em [01-automacao/interface-n8n.md](../01-automacao/interface-n8n.md) para consulta futura.

**Etapa 1 — Fundamentos de automação: concluída. ✅**

---

## 02/11/2026 — Etapa 2, Aula 02: Webhooks e fundamentos de API REST

> Conteúdo e exercícios em [02-apis-e-integracoes/aula-02-webhooks-e-api-rest.md](../02-apis-e-integracoes/aula-02-webhooks-e-api-rest.md).
>
> **Praticado de fato em 04/11** (troca de PC entre 02 e 03/11 atrasou a execução — conteúdo já estava documentado, só faltou a prática).

### Conceitos

- **Webhook** — trigger que inicia o workflow quando recebe uma requisição HTTP externa, ao contrário do Schedule/Manual Trigger.
- **Diferença entre HTTP Request e Webhook (cliente vs. servidor)** — no HTTP Request o n8n é cliente (consulta uma API externa); no Webhook o n8n é servidor (expõe uma URL que outros sistemas chamam).
- **Segurança básica de webhooks** — Test URL só funciona uma vez por clique em "Listen"; em produção, validar sempre o dado recebido e considerar autenticação.

### Exercício 1 — Primeiro Webhook (Postman → n8n)

Testado com Postman em vez de curl. Fluxo: configurar node Webhook (POST) → copiar Test URL → "Listen for test event" → enviar POST no Postman com body JSON (`nome`, `email`, `idade`).

**Erro encontrado:** na primeira tentativa, o `body` chegou como string bruta (`"{\n  \"nome\": ...}"`), com header `content-type: text/plain`. Causa: no Postman, Body → raw estava com o dropdown em "Text" em vez de "JSON", então o Content-Type não foi setado corretamente.

**Correção:** trocado o dropdown pra "JSON" no Postman. Resultado: `body` chegou como objeto JSON de verdade (`content-type: application/json`), com `nome`, `email` e `idade` acessíveis diretamente via `{{ $json.body.idade }}`.

### Exercício 2 — Webhook + validação (If)

Reaproveitada a lógica do If (`idade > 19`) já validada na Aula 01, trocando a expressão para `{{ $json.body.idade }}` (dado agora chega aninhado em `body`). Testado com `idade: 10` via Postman → caiu corretamente no **False Branch**. Confirma que a lógica de condição independe da origem do dado (manual, API externa ou evento via Webhook).

### Exercício 3 — Desafio: Webhook + HTTP Request (mini-TechPulse orientado a evento)

Workflow duplicado a partir do anterior (pra preservar os Exercícios 1/2) e montado: **Webhook (recebe `title`, `url`) → HTTP Request (GET api.agify.io, simulando enriquecimento) → Edit Fields (combina os dois)**.

**Erros encontrados e corrigidos:**
1. Tentativa inicial no modo "JSON" do Edit Fields sem sintaxe de objeto válida (faltavam chaves, aspas e vírgulas) → erro de validação.
2. Ao trocar para modo "Fixed"/Manual Mapping, o nome do campo e a expressão foram digitados juntos na mesma caixa (`titulo: {{ ... }}`) — o n8n entendeu isso como nome de campo e trocou sozinho pra aba "Expression", deixando o valor vazio. Corrigido separando: caixa de **nome** só com `titulo`, caixa de **value** com a expressão.
3. Resquício do erro anterior: sobrou um `:` no final dos nomes dos campos (`"titulo:"`, `"url:"`, `"relevancia_estimada:"`). Corrigido removendo o `:` de cada nome.

**Resultado final:**
```json
{
  "count": 439,
  "name": "ia",
  "age": 54,
  "titulo": "iFood lança novo modelo de IA",
  "url": "https://exemplo.com/noticia",
  "relevancia_estimada": "54"
}
```

Combinação de dado do Webhook (2 nodes atrás, via `$('Webhook').item.json...`) com dado do HTTP Request (node imediatamente anterior, via `$json...`) funcionando no mesmo item.

### Dificuldades encontradas

- Confundir o Content-Type de uma requisição manual (Postman) é um erro comum e realista — o mesmo aconteceria com qualquer cliente HTTP mal configurado. Boa lição sobre nunca assumir o formato do dado recebido sem checar.
- No Edit Fields, misturar nome do campo com a expressão no mesmo input, e diferença entre os modos "Fixed" e "JSON" do node — cada um exige uma sintaxe diferente.
- Entender que `$json` sempre refere ao node imediatamente anterior, e que pra pegar dado de um node mais atrás no fluxo é preciso referenciar pelo nome com `$('NomeDoNode')`.

### Aprendizados e próximos passos

Entendido o papel do Webhook como "porta de entrada" de eventos externos, a importância do header `Content-Type`, e — o aprendizado mais valioso do dia — como combinar dados de múltiplos nodes anteriores (não só o imediato) dentro de um mesmo item, técnica essencial para o pipeline do TechPulse (coletar → enriquecer → organizar).

**Etapa 2 — Lógica e integração de APIs: concluída. ✅**

---

**Retomar depois:** seguir para Etapa 3 — Integração e orquestração de IA (06/11).

## 04/11/2026 (cont.) — Etapa 3, Aula 03: RSS e primeiros passos com Gemini

> Conteúdo e exercícios em [03-orquestracao-ia/aula-03-rss-e-gemini.md](../03-orquestracao-ia/aula-03-rss-e-gemini.md). Adiantado em relação ao cronograma (previsto para 06/11).

### Conceitos

- **RSS Feed Read** — consulta uma fonte RSS sob demanda; retornou 21 itens de um feed real, cada um com `title`, `link`, `pubDate`, `content`/`contentSnippet`.
- **Diferença entre API "de dado" e API "de IA" (prompt → resposta variável)** — API de dado (agify) sempre retorna o mesmo formato previsível; API de IA (Gemini) recebe um prompt e devolve texto gerado, variável a cada chamada, dentro de um "envelope" JSON (`candidates[0].content.parts[0].text`).
- **Credencial Header Auth (Gemini API key)** — guardada via Settings → Credentials → Header Auth, nunca exposta no node.

### Exercício 1 — Ler fonte RSS real

Configurado RSS Feed Read com feed real (feedburner/TechCrunch). 21 itens retornados, cada um com título, link, data e conteúdo (HTML bruto em `content`, versão limpa em `contentSnippet`).

### Exercício 2 — Enviar notícia pro Gemini

**Erros encontrados e corrigidos, em sequência:**
1. Campo "Name" do header da credencial preenchido com o nome/rótulo da credencial em vez do nome real do header (`x-goog-api-key`) → corrigido editando o campo certo dentro da credencial.
2. "Send Body" desligado → corpo vazio, Gemini reclamando de `contents` não especificado → ligado e configurado em modo JSON.
3. Uso do campo `content` (HTML bruto, cheio de aspas) dentro do prompt quebrou o JSON por aspas não escapadas → trocado para usar só `title` no prompt.
4. Modelo usado: `gemini-3.8-flash` (confirmado como nome atual, substituiu `gemini-3.7-flash` em out/2026).
5. Múltiplos **503 (Service Unavailable)** e um **429 (rate limit)** do Gemini durante os testes — identificados via log do container Docker (`docker compose logs n8n`), já que a chamada roda no backend e não aparece no DevTools do navegador.

### Exercício 3 — Organizar resultado + padrão de fallback

Ao final, decidimos ir além do exercício original e implementar um **padrão de fallback**: HTTP Request principal (`gemini-3.8-flash`) com **Settings → On Error: "Continue (using error output)"**, output de erro conectado a um segundo HTTP Request (fallback, modelo alternativo), e os outputs de sucesso de ambos convergindo no mesmo node **If**.

Armadilha no caminho: tentativa inicial de criar o "If" renomeando um node **Edit Fields** — não funciona, o tipo do node não muda por renomear; foi preciso adicionar um node **If** de verdade.

**Validação estrutural:** confirmado visualmente que a arquitetura está correta (Success → If direto; Error → fallback → If). Testes end-to-end ficaram inconclusivos porque, numa das rodadas, até o modelo de fallback tomou 503 — instabilidade da API gratuita do Gemini no momento, não erro de configuração.

Também ativada a opção **"Execute Once"** no RSS Read, para processar só o primeiro item durante os testes em vez dos 21 de uma vez — reduz o consumo de cota da API enquanto se depura o workflow.

### Dificuldades encontradas

- Diferenciar o "nome da credencial" (rótulo na lista) do "nome do header" (exigido pela API) — dois campos parecidos, fácil de confundir.
- JSON montado por concatenação de string quebra com dado que contém aspas (HTML do RSS).
- Node renomeado não muda de tipo — "If" só existe adicionando o node certo.
- Depurar um node HTTP Request exige olhar o log do backend (Docker), não o DevTools do navegador — a chamada não passa pelo browser.

### Aprendizados e próximos passos

Dia mais denso em debugging real do que os anteriores — quase todos os erros foram de integração (headers, JSON, tipos de node), não de lógica. Also implementado, por iniciativa própria, um padrão de resiliência (fallback entre dois modelos) que vai além do que a aula pedia.

**Etapa 3 (parcial) — RSS + Gemini: arquitetura validada, pendente só reexecutar quando a API do Gemini estabilizar.**

**Próximo passo:** retestar o fluxo completo depois de uma pausa, depois seguir para os próximos dias da Etapa 3 (construção de workflows com processamento por IA, engenharia de prompts, orquestração de múltiplas etapas).
