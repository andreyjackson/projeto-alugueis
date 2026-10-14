# Gestão de Lojistas | Galpão 47

Painel web de demonstração para acompanhar a operação de um galpão comercial e a gestão de seus lojistas. A interface reúne indicadores, ocupação, contratos, financeiro, relacionamento e análises em uma navegação lateral.

> **Estado do projeto:** protótipo em evolução, construído com Nuxt. As telas são renderizadas no servidor (SSR) e organizadas por domínio. O domínio de contratos já consome uma API REST do próprio projeto (`/api/contracts`); os demais usam dados de demonstração tipados. Ainda não há banco de dados nem autenticação.

## Funcionalidades

O menu do painel está organizado em três áreas:

- **Operação:** dashboard executivo, visão geral, lojistas, portfólio de espaços, mapa do galpão, contratos e financeiro.
- **Relacionamento:** leads de locação, comunicados, manutenção e eventos.
- **Análise:** desempenho, relatórios e fluxo de visitantes.

A rota `/` redireciona para o dashboard executivo, e rotas desconhecidas também voltam para ele. Em contratos, o filtro de status consulta a API. Busca, envio de comunicados, cadastros e exportações ainda são apenas visuais.

## Tecnologias

- [Nuxt 4](https://nuxt.com) (Vue 3, Vue Router, servidor Nitro)
- TypeScript em modo estrito, com checagem de templates via `vue-tsc`
- Vitest com `@nuxt/test-utils`, `@vue/test-utils` e `happy-dom`
- Oxlint para análise estática
- CSS puro, sem framework

## Requisitos

- Node.js `^22.19`, `^24.11` ou `>=26` (exigência do Nuxt 4)
- npm

## Inicialização

```bash
npm install     # também gera os tipos do Nuxt em .nuxt/ (postinstall)
npm run dev
```

Abra `http://localhost:3000`.

## Comandos disponíveis

| Comando | Descrição |
| --- | --- |
| `npm run dev` | Servidor de desenvolvimento com atualização automática. |
| `npm run build` | Build de produção em `.output/`. Execute com `node .output/server/index.mjs`. |
| `npm run generate` | Gera um site estático (não inclui a API do servidor). |
| `npm run preview` | Serve localmente o build de produção. |
| `npm run lint` | Analisa o código com Oxlint. |
| `npm run typecheck` | Verifica os tipos, incluindo os templates `.vue`. |
| `npm run test` | Executa os testes unitários e os testes de tela. |

## Arquitetura

```text
.
├── nuxt.config.ts           # Configuração do Nuxt (CSS global, head, redirecionamentos)
├── vitest.config.ts         # Projetos de teste: unit (Node) e nuxt (app simulada)
├── app/                     # Aplicação Vue (srcDir do Nuxt)
│   ├── app.vue              # Raiz: layout + página atual
│   ├── layouts/default.vue  # Shell: sidebar, topbar e título da aba
│   ├── pages/               # Uma página por rota (<key>.vue → /<key>)
│   ├── components/
│   │   ├── AppSidebar.vue   # Menu lateral
│   │   ├── AppTopbar.vue    # Cabeçalho com data, busca e ações
│   │   └── ui/              # Componentes visuais reutilizáveis (<UiPanel>, <UiMetricGrid>...)
│   ├── features/<domínio>/  # Dados, tipos, composables e componentes de cada domínio
│   ├── config/navigation.ts # Itens do menu lateral
│   ├── data/                # Mocks usados por mais de um domínio
│   ├── types/ui.ts          # Tipos visuais compartilhados (Tone, Metric, StatusListItem...)
│   ├── utils/               # Utilitários (formatação de datas)
│   └── assets/css/          # CSS global e do painel
├── server/                  # API (Nitro)
│   ├── api/contracts.get.ts # GET /api/contracts?status=...
│   └── data/                # Dados de demonstração do servidor
├── shared/                  # Regras de domínio usadas pelo app e pelo servidor
└── test/nuxt/               # Testes de tela no ambiente Nuxt
```

### Convenções

- **Componentes de `app/components/` são importados automaticamente.** Os de `ui/` recebem o prefixo `Ui` (ex.: `ui/Panel.vue` → `<UiPanel>`).
- **Código de domínio é importado explicitamente** a partir de `~/features/<domínio>/...`. Quando um domínio é usado por outro, ele expõe sua API pública num `index.ts` (ex.: `~/features/lojistas`).
- **`shared/` não depende de Vue nem do servidor**: tipos e regras puras (filtros, resumos, validação), importados via `#shared/...`.
- Rótulos e cores de status ficam no app (ex.: `features/contratos/contracts.ui.ts`), não em `shared/`.

### Como adicionar uma tela

1. Crie `app/pages/<key>.vue` e, se precisar de dados, `app/features/<key>/<key>.data.ts`.
2. Monte a tela com os componentes `Ui*`.
3. Adicione o item em `app/config/navigation.ts`.

O teste `test/nuxt/pages.nuxt.test.ts` falha se houver página sem item no menu (ou o contrário) e renderiza cada tela dentro do layout.

### Dados e integração com backend

`contratos` é o modelo para domínios com dados reais:

1. `shared/contracts.ts`: tipos e regras (`filterContracts`, `getContractSummary`).
2. `server/api/contracts.get.ts`: endpoint que aplica as regras sobre a fonte de dados.
3. `app/features/contratos/useContracts.ts`: composable com `useFetch`, que recarrega quando o filtro muda.
4. `app/pages/contratos.vue`: tela que consome só o composable.

Para usar um banco de dados, troque a fonte em `server/` (hoje `server/data/contracts.mock.ts`); a tela e o composable não mudam. Os demais domínios migram para esse padrão movendo seus `*.data.ts` para uma rota em `server/api/`.

## Testes

- **unit** (`*.test.ts` em `app/`, `shared/`, `server/`): regras puras em Node.
- **nuxt** (`*.nuxt.test.ts`): telas montadas numa aplicação Nuxt simulada, com endpoints mockados via `registerEndpoint`.
# projeto-alugueis
