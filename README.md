# Preparação para o Programa de Estágio em GenAI — iFood

Repositório dedicado à minha preparação técnica para o Programa de Estágio em GenAI do iFood, com foco na construção de soluções utilizando Inteligência Artificial, automação de processos, integração de APIs e orquestração de workflows.

O objetivo é transformar o aprendizado teórico em experiências práticas, desenvolvendo soluções que utilizem IA para resolver problemas reais.

## Objetivos

- Compreender os fundamentos de automação e orquestração de processos.
- Desenvolver habilidades com ferramentas como n8n.
- Aprender a consumir, integrar e manipular APIs.
- Explorar a integração de modelos de linguagem (LLMs) em aplicações.
- Desenvolver workflows inteligentes e automatizados.
- Compreender os conceitos de agentes de IA e Function Calling.
- Construir projetos práticos que demonstrem a aplicação desses conhecimentos.

## Trilha de estudos e documentação

> 📖 Acompanhe o registro detalhado de cada experimento, erros enfrentados e decisões técnicas no **[Diário de Aprendizado](docs/diario-de-aprendizado.md)** e o cronograma completo no **[Plano de Estudos](docs/plano-de-estudos.md)**.

| Etapa | Conteúdo | Materiais e Práticas | Status |
|---|---|---|---|
| 01 | Fundamentos de automação | [Aula 01 — Fundamentos](01-automacao/aula-01-fundamentos.md) · [Interface do n8n](01-automacao/interface-n8n.md) | 🟢 Concluído |
| 02 | n8n e construção de workflows | Condicionais (`IF`), manipulação JSON e expressões | 🟢 Concluído |
| 03 | APIs, HTTP e Webhooks | [Aula 02 — Webhooks e API REST](02-apis-e-integracoes/aula-02-webhooks-e-api-rest.md) | 🟢 Concluído |
| 04 | Integração e orquestração de IA | [Aula 03 — RSS e Gemini](03-orquestracao-ia/aula-03-rss-e-gemini.md) · [Aula 04 — Prompts e JSON](03-orquestracao-ia/aula-04-prompts-e-json-estruturado.md) · [Aula 05 — Orquestração](03-orquestracao-ia/aula-05-orquestracao-multiplas-etapas.md) | 🟡 Em andamento |
| 05 | Agentes de IA e Function Calling | Agentes, ferramentas e validação | ⚪ Não iniciado |
| 06 | Projeto prático — TechPulse | Integração end-to-end do MVP | ⚪ Não iniciado |
| 07 | Preparação técnica para entrevistas | Revisão conceitual e portfólio | ⚪ Não iniciado |

## Evolução prática no n8n (TechPulse Pipeline)

Durante as etapas de automação e orquestração de IA, o pipeline do **TechPulse** foi evoluído incrementalmente no **n8n** (self-hosted via Docker):

### Etapa 3 — Orquestração com IA, Fallback de Modelos, JSON Estruturado e Agregação
Integração de coleta de feeds RSS, controle de lote (`Limit`), classificação e resumo com a **API do Google Gemini** (`responseMimeType: "application/json"`), arquitetura de **fallback automático entre modelos** para resiliência contra limites de cota (`429`/`503`), curadoria semântica condicional (`If`) e consolidação de boletins (`Aggregate`):

![Workflow TechPulse — Orquestração em múltiplas etapas](docs/assets/workflow-etapa3-orquestracao.png)

<details>
<summary><strong>📸 Ver workflows das Etapas 1 e 2 (Fundamentos, APIs REST e Webhooks)</strong></summary>

<br>

**Etapa 1 — Consumo de API REST (`api.agify.io`), manipulação de campos e ramificação condicional (`IF`):**
![Workflow Etapa 1 — HTTP Request e Condicional IF](docs/assets/workflow-etapa1-fundamentos-api.png)

**Etapa 2 — Recepção de eventos via Webhook (`POST`) e enriquecimento de dados com API externa:**
![Workflow Etapa 2 — Webhook e Enriquecimento de Dados](docs/assets/workflow-etapa2-webhook-enriquecimento.png)

</details>

## Projetos e experimentos

### LinkedIn AI Post Generator

Aplicação desenvolvida em Python e Streamlit, integrada à API do Google Gemini, para auxiliar na geração estruturada de publicações para o LinkedIn.

**Tecnologias:** Python, Streamlit, Google Gemini API e JSON.

### TechPulse

Projeto em desenvolvimento conceitual, planejado para automatizar a coleta, classificação e síntese de notícias sobre tecnologia.

A proposta é integrar fontes RSS, processamento com IA e armazenamento de informações em uma plataforma de acompanhamento tecnológico.

**Tecnologias planejadas:** React, TypeScript, Node.js, MongoDB, Gemini API e n8n.

### GitQuest

Aplicação educacional desenvolvida com HTML, CSS e JavaScript, com o objetivo de ensinar conceitos e operações do Git por meio de uma experiência interativa e gamificada.

## Organização do repositório

```text
preparacao-estagio-ifood/
├── README.md
├── compose.yml
├── docs/
│   ├── assets/
│   ├── plano-de-estudos.md
│   ├── diario-de-aprendizado.md
│   └── referencias.md
├── 01-automacao/
│   ├── aula-01-fundamentos.md
│   └── interface-n8n.md
├── 02-apis-e-integracoes/
│   └── aula-02-webhooks-e-api-rest.md
├── 03-orquestracao-ia/
│   ├── aula-03-rss-e-gemini.md
│   ├── aula-04-prompts-e-json-estruturado.md
│   └── aula-05-orquestracao-multiplas-etapas.md
├── 04-agentes-ia/
├── 05-projeto-pratico/
└── 06-preparacao-entrevista/
```

A estrutura será ampliada conforme os estudos e experimentos evoluírem.

### Rodando o n8n localmente

O ambiente de automação (n8n) roda via Docker Compose:

```bash
docker compose up -d
```

A interface fica disponível em `http://localhost:5678`.

## Metodologia

A preparação será organizada em três frentes:

- **Teoria:** compreensão dos conceitos e fundamentos técnicos.
- **Prática:** construção de workflows, experimentos e aplicações.
- **Documentação:** registro dos aprendizados, dificuldades, decisões técnicas e resultados.

Cada atividade relevante deverá registrar:

1. Objetivo.
2. Conceitos estudados.
3. Aplicação prática.
4. Dificuldades encontradas.
5. Soluções desenvolvidas.
6. Aprendizados e próximos passos.

## Acompanhamento

O progresso será atualizado conforme a conclusão das atividades, mantendo uma distinção entre conteúdos estudados, experimentos realizados e funcionalidades efetivamente implementadas.

---

**Objetivo final:** desenvolver a capacidade de construir soluções inteligentes, integrando software, automação e Inteligência Artificial para resolver problemas de maneira prática e eficiente.

## Contato

**Vinicius Menegussi** — [LinkedIn](https://www.linkedin.com/in/vinicius-menegussi-dev)