# Aula 01 — Fundamentos de Automação

**Trilha:** Automação e Workflows
**Ferramenta:** n8n
**Status:** Fundamentos estudados e exercícios de modelagem concluídos.

## 1. Objetivo da aula

Compreender os fundamentos de automação de processos, identificar os principais componentes de um workflow e entender como diferentes etapas podem ser conectadas para executar tarefas automaticamente.

## 2. Conceitos fundamentais

### Workflow

Um workflow é um fluxo de trabalho composto por etapas conectadas, organizadas para executar um determinado processo.

Cada etapa desempenha uma função dentro da automação.

### Node

Um node é uma unidade individual dentro de um workflow. Cada node executa uma operação específica, como consultar uma API, verificar uma condição ou enviar uma mensagem.

### Trigger

É o gatilho responsável por iniciar a execução de um workflow.

Exemplos:
- **Schedule Trigger:** inicia a execução em horários ou intervalos definidos.
- **Manual Trigger:** permite iniciar a execução manualmente.
- **Webhook:** inicia o fluxo quando recebe uma requisição externa.
- **RSS Trigger:** pode iniciar o fluxo quando detecta novas publicações em um feed.

### Action

É uma operação executada durante o workflow, como consultar uma API, processar informações, enviar um e-mail ou armazenar dados.

### IF

O node IF avalia uma condição e direciona a execução para diferentes caminhos.

- **True:** condição satisfeita.
- **False:** condição não satisfeita.

É utilizado para implementar decisões dentro de uma automação.

### JSON

JSON é um formato de representação e troca de dados utilizado frequentemente na comunicação entre sistemas e APIs.

Exemplo:

```json
{
  "title": "Novidades em Inteligência Artificial",
  "category": "Tecnologia",
  "relevant": true
}
```

### Expressões no n8n

As expressões permitem acessar e utilizar informações recebidas de outros nodes.

Exemplo:

```javascript
{{ $json.title }}
```

A expressão acessa a propriedade `title` do objeto JSON de entrada.

### HTTP Request

Node utilizado para realizar requisições HTTP e integrar o workflow com APIs externas.

Principais métodos HTTP:

| Método | Finalidade geral |
|---|---|
| GET | Consultar informações |
| POST | Enviar ou criar informações |
| PUT | Substituir um recurso |
| PATCH | Atualizar parcialmente um recurso |
| DELETE | Remover um recurso |

### API REST

Uma API REST permite que diferentes aplicações se comuniquem por meio de requisições HTTP, utilizando endpoints e formatos de dados como JSON.

### n8n

O n8n é uma plataforma de automação e orquestração de workflows. Ele permite conectar aplicações, serviços, APIs e modelos de Inteligência Artificial para construir processos automatizados.

O n8n não é um modelo de linguagem. Ele pode integrar modelos como o Gemini para adicionar capacidades de IA aos workflows.

## 3. Diferença entre automação tradicional e integração com IA

Uma automação tradicional executa operações e decisões previamente definidas.

Exemplo:
- Receber um cadastro.
- Verificar se o e-mail está preenchido.
- Direcionar o fluxo conforme a condição.
- Armazenar os dados.

Quando integramos um modelo de IA, podemos adicionar operações como classificação de textos, geração de resumos e interpretação de informações.

Exemplo:
- Receber uma notícia.
- Enviar seu conteúdo para um modelo de IA.
- Classificar o assunto.
- Gerar um resumo.
- Armazenar o resultado.

## 4. Exercícios de modelagem

### Exercício 01 — Automação de notícias (TechPulse)

**Objetivo:** projetar um workflow para coletar e processar notícias tecnológicas.

Fluxo proposto:

1. Schedule Trigger — inicia a execução diariamente às 8h.
2. RSS Feed Read — consulta uma fonte RSS.
3. Filter — seleciona notícias relevantes.
4. Gemini — classifica as notícias e gera resumos.
5. Database — armazena os resultados.

**Aprendizado:** compreender a sequência de execução de um workflow e a diferença entre um gatilho baseado em horário e um node que consulta uma fonte RSS.

**Observação:** o RSS Feed Read é adequado quando a coleta é iniciada por um agendamento. O RSS Trigger é uma alternativa quando queremos iniciar o workflow a partir da detecção de novas publicações.

### Exercício 02 — Automação de cadastro de clientes

**Objetivo:** projetar um workflow que valide os dados de um formulário e direcione o cadastro conforme a presença e a validade do e-mail.

Fluxo proposto:

1. Form Trigger ou Webhook — recebe os dados do formulário.
2. IF — verifica se o e-mail foi preenchido e possui formato válido.
3. Caminho True:
   - Armazena os dados do cliente.
   - Define o status do cadastro.
   - Envia uma confirmação por e-mail.
4. Caminho False:
   - Registra o cadastro como pendente.
   - Retorna uma mensagem solicitando um e-mail válido.

**Aprendizado:** compreender validação de dados, ramificações condicionais e persistência de informações.

**Observação:** o Manual Trigger pode ser utilizado para testar o workflow, mas em uma aplicação real o formulário deve iniciar a automação por meio de um Form Trigger ou Webhook.

## 5. Aprendizados da aula

- Compreensão da estrutura de um workflow.
- Identificação das responsabilidades de triggers e actions.
- Utilização de condições para direcionar processos.
- Noções de manipulação de dados JSON.
- Entendimento da comunicação entre sistemas por APIs.
- Compreensão do papel do n8n na automação e integração de serviços.
- Identificação de possibilidades de integração de modelos de IA em processos automatizados.

## 6. Próximos passos

- Construir os primeiros workflows no n8n.
- Explorar a interface e a configuração dos nodes.
- Executar fluxos manualmente.
- Trabalhar com dados de entrada e saída.
- Implementar condições utilizando o node IF.

---

**Conclusão:** a aula apresentou os fundamentos necessários para compreender como processos podem ser divididos em etapas, conectados e automatizados por meio de workflows.
