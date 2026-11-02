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

### Conceitos

- **Webhook** —
- **Diferença entre HTTP Request e Webhook (cliente vs. servidor)** —
- **Segurança básica de webhooks (validação, autenticação)** —

### Exercício 1 — Primeiro Webhook (curl → n8n)

_(preencher)_

### Exercício 2 — Webhook + validação (If)

_(preencher)_

### Exercício 3 — Desafio (opcional)

_(preencher, se feito)_

### Dificuldades encontradas

### Aprendizados e próximos passos

---

**Retomar depois:** seguir para Etapa 3 — Integração e orquestração de IA.
