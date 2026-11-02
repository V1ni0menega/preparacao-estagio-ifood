# Referência rápida — Interface do n8n

Guia de consulta rápida sobre onde fica cada coisa na interface do n8n. Útil para voltar aqui quando precisar, em vez de decorar.

## Barra lateral esquerda

- **Assistant** — IA integrada do próprio n8n, ajuda a construir/debugar workflows.
- **Overview** — lista todos os workflows existentes no workspace.
- **Templates** — biblioteca de workflows prontos da comunidade, boa fonte de inspiração (ex.: buscar "RSS" ou "AI").
- **Insights** — métricas de execução (sucessos, falhas, tempo), mais útil com vários workflows rodando.
- **Settings → Credentials** — onde ficam guardadas as chaves de API (API keys, tokens, OAuth) de forma segura, sem expor nos nodes. Usar sempre que um node precisar de autenticação.

## Barra superior (dentro de um workflow)

- **Editor** — onde o workflow é construído (tela padrão).
- **Executions** — histórico de todas as execuções do workflow, com o que passou e o que falhou em cada uma. Principal ferramenta de debug.
- **Evaluations** — testes automatizados do workflow (uso mais avançado).
- **Publish** — ativa o workflow para rodar sozinho via trigger automático (sem precisar clicar em "Execute workflow" manualmente).

## Canvas

- Ícones no canto superior direito: zoom in/out, busca de nodes, "tidy up" (organiza layout automaticamente), fullscreen.

## Dentro de um node (painel de detalhe)

- **Input / Output**, com abas **Schema / Table / JSON** — formas diferentes de visualizar o mesmo dado que entra/sai do node.
- **Execute step** — roda só aquele node isoladamente, sem rodar o workflow inteiro (ótimo para testar e debugar passo a passo, como fizemos no If e no HTTP Request).
- **Docs** (canto superior direito do node) — leva direto pra documentação oficial daquele node específico.
