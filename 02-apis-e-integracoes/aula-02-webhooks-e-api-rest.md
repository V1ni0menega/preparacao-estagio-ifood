# Aula 02 — Webhooks e Fundamentos de API REST

**Trilha:** Automação e Workflows
**Ferramenta:** n8n
**Status:** Concluída (em 04/11/2026).

## 1. Objetivo da aula

Entender como um sistema externo pode **iniciar** um workflow em tempo real (ao contrário do Schedule Trigger, que roda em horários fixos), e revisar os fundamentos de API REST aplicados na prática do dia anterior.

## 2. Recapitulando API REST (consolidação da Aula 01)

Já praticamos isso ao consumir `api.agify.io` com o **HTTP Request**: seu workflow fez o papel de **cliente** de uma API REST.

Hoje a ideia se inverte: com o **Webhook**, o seu workflow n8n vira o **servidor** — ele expõe um endereço (URL) que outros sistemas podem chamar.

| | HTTP Request (Aula 01) | Webhook (Aula 02) |
|---|---|---|
| Papel do n8n | Cliente (consome uma API) | Servidor (expõe uma API) |
| Quem inicia | O workflow, quando executa | Um sistema externo, quando chama a URL |
| Analogia | Você liga para alguém | Alguém te liga |

## 3. O que é um Webhook

Um Webhook é um **trigger que inicia o workflow quando recebe uma requisição HTTP externa** (geralmente POST, com um corpo em JSON).

Exemplos reais de uso:
- Um formulário de cadastro no site chama o webhook para processar o cadastro.
- Um serviço de pagamento avisa "pagamento aprovado" chamando um webhook.
- O GitHub chama um webhook sempre que alguém faz um push (é assim que CI/CD funciona).

### Anatomia de uma chamada de Webhook

```
POST https://seu-n8n.com/webhook/cadastro-cliente
Content-Type: application/json

{
  "nome": "Vinicius",
  "email": "vinicius@exemplo.com",
  "idade": 29
}
```

O n8n recebe essa requisição, transforma o corpo (body) em dado JSON disponível para os próximos nodes — exatamente como a resposta de uma API em um HTTP Request.

### Segurança básica

- Um Webhook em modo "Test" só funciona uma vez e expõe uma URL temporária — bom para desenvolvimento.
- Em produção (workflow publicado), a URL fica fixa — por isso é importante **validar os dados recebidos** (nunca confiar cegamente no que chega) e, quando possível, usar autenticação (header com token, por exemplo).

## 4. Exercícios práticos

### Exercício 1 — Primeiro Webhook

1. Crie um novo workflow.
2. Adicione um node **Webhook** como trigger.
3. Configure o método como `POST` e copie a "Test URL".
4. Clique em "Execute workflow" / "Listen for test event" (deixa o n8n esperando a chamada).
5. Em outra aba/terminal, dispare uma requisição para essa URL. Pode ser com `curl`:

```bash
curl -X POST <test-url-do-webhook> \
  -H "Content-Type: application/json" \
  -d '{"nome": "Vinicius", "email": "vinicius@exemplo.com", "idade": 29}'
```

6. Confirme no n8n que os dados chegaram no node Webhook (aba Output).

### Exercício 2 — Webhook + validação (reaproveitando a Aula 01)

Monte: **Webhook → If (idade > 19) → Edit Fields (status: aprovado | pendente)**.

É o mesmo fluxo de condição que você já construiu — a única mudança é a origem do dado: antes vinha de um Edit Fields manual, depois de uma API externa (HTTP Request), e agora chega "de fora" via Webhook, em tempo real.

**Aprendizado esperado:** perceber que, uma vez que o dado vira JSON dentro do n8n, a lógica de condição (IF) não muda — o que muda é só o trigger que inicia o processo.

### Exercício 3 — Desafio (opcional, se sobrar tempo)

Combine os dois conceitos: um Webhook que recebe uma notícia (`title`, `url`) e, antes de salvar, faz um **HTTP Request** para enriquecer o dado (ex.: consultar outra API). Isso é essencialmente um mini-TechPulse orientado a evento em vez de agendado.

## 5. Antes de trocar de computador

O workflow construído fica salvo só no volume Docker local (`n8n_data`), **não no Git**. Para continuar de onde parou em outro PC:

1. No n8n, abra o workflow e use **"..." → Download** (ou `Export`) para baixar o JSON do workflow.
2. Salve esse arquivo em `01-automacao/workflows/` neste repositório (crie a pasta se não existir).
3. Faça commit e push.
4. No outro PC: `git clone`, depois `docker compose up -d`, abra `localhost:5678` e use **Import from File** para subir o mesmo workflow.

## 6. O que deve ficar no diário

Registrar, como nas aulas anteriores: diferença entre Webhook e outros triggers, resultado dos testes (curl → n8n), e qualquer dificuldade com a URL de teste ou validação dos dados.
