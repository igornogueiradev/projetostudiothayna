# Studio Thayna — Contexto do Projeto

## Visão Geral

PWA (Progressive Web App) de gestão para estúdio de beleza — agendamento, controle de equipe, financeiro e integração com Google Calendar. Aplicação single-page em HTML/CSS/JS puro, sem frameworks, hospedada na Vercel.

**URL de produção:** `https://projetostudiothayna.vercel.app`
**Repositório:** `https://github.com/igornogueiradev/projetostudiothayna`

> **Deploy:** o GitHub integration do Vercel não detecta commits automaticamente — sempre usar `vercel --prod --yes` após `git push origin main`.

---

## Arquitetura

| Arquivo | Papel |
|---|---|
| `index.html` | Toda a aplicação: HTML + CSS + JS (~2972 linhas) |
| `studio-thayna.html` | Versão/cópia alternativa do arquivo principal |
| `logo.png` | Logotipo do estúdio |
| `vercel.json` | Configuração de deploy (todas as rotas → `index.html`) |
| `.vercel/project.json` | IDs do projeto Vercel (`prj_WHKUb9EWrvaL5dvXE9A600Fv2Ht6`) |

### Backend / Serviços externos

- **Firebase Firestore** — banco de dados principal (coleções: `eventos`, `clientes`, `colaboradoras`, `servicos`, `pagamentos_confirmados`)
- **Firebase Auth** — autenticação anônima (sem login do usuário)
- **Google Calendar API** — integração bidirecional via OAuth2 implicit flow
- **Fontes:** Google Fonts (Playfair Display + DM Sans)

### Configurações Firebase (já no código)
```
projectId: studio-thayna
authDomain: studio-thayna.firebaseapp.com
appId: 1:219805696286:web:be3c087a77269ca94a490f
```

### Google OAuth
```
CLIENT_ID: 1054000419123-63epgfgkac3an09skhiq7tpf3861me4n
API_KEY: AIzaSyDzF35wR7guepU4CtsMLL9WS4BdhHsSzbA
SCOPE: https://www.googleapis.com/auth/calendar
```
Token armazenado em `localStorage` com TTL; fluxo implicit grant (redirect + hash).

---

## Telas e Funcionalidades

### 1. Agenda (`#tela-agenda`)
- Calendário mensal com navegação por mês
- Dias com eventos marcados com ponto indicador
- Ao clicar em um dia: lista de eventos do dia + botão "novo evento"
- Sincronização com Google Calendar (todos os calendários da conta)

### 2. Eventos (`#tela-eventos`)
- Lista de todos os eventos com filtro por status (todos / pendente / confirmado / concluído)
- Card com nome, data, hora, local, badge de status, contagem de atendimentos, valor recebido/total
- Modal de detalhe: resumo financeiro, status editável, lista de atendimentos com pagamentos
- Criação/edição via modal com campos: nome, data, hora, local, colaboradoras, clientes, cronograma, deslocamento

### 3. Equipe (`#tela-colaboradoras`)
- **Aba Colaboradoras:** cadastro com nome, telefone, comissão padrão (%), chave PIX
- **Aba Serviços:** cadastro de serviços disponíveis (nome)
- FAB (+) cria colaboradora ou serviço conforme aba ativa

### 4. Financeiro (`#tela-financeiro`)
- Filtro por mês (seletor)
- Cards de resumo: total recebido, saldo líquido, comissões, a receber
- Lista de comissões a pagar / pagas com chave PIX para copiar
- Histórico por colaboradora (barras)
- Distribuição por forma de pagamento (PIX, crédito, débito, dinheiro, transferência)

---

## Modais

| ID | Função |
|---|---|
| `modal-evento` | Criar/editar evento |
| `modal-detalhe-evento` | Visualizar detalhes do evento |
| `modal-cliente` | Cadastrar cliente |
| `modal-colaboradora` | Cadastrar/editar colaboradora |
| `modal-servico` | Cadastrar/editar serviço |
| `modal-sel-servicos` | Selecionar serviços de um atendimento |
| `modal-pagamento` | Registrar pagamento (sinal / restante / total) |
| `modal-slot-cronograma` | Editar slot individual do cronograma |
| `modal-linha-tempo` | Cronograma por cliente (linha do tempo) |
| `modal-grade-interativa` | Grade interativa drag-and-drop; exporta para Excel |

---

## Estado Global (`state`)

```js
state = {
  telaAtual, mesAtual, diaSelecionado,
  googleToken,
  colabsEvento, atendimentosEvento, duracaoServicosEvento, duplasEvento,
  eventoEditandoId, colabEditandoId, selServicosAtendIdx, eventoDetalheId,
  clientes, colaboradoras, eventos, servicos,
  pagamentosConfirmados, googleIdsNoSupabase,
  _equipeAba
}
```

---

## Funcionalidades Especiais

- **Importação via WhatsApp:** colar lista de clientes no formato `Nome - Serviço - Horário` para criar atendimentos em lote
- **Distribuição automática:** `distribuirAutomatico()` distribui clientes entre colaboradoras automaticamente
- **Grade interativa:** visualização drag-and-drop com seleção múltipla por coluna; exporta para Excel via SheetJS
- **Cronograma por cliente:** linha do tempo visual por colaboradora
- **Deslocamento:** campo opcional por evento, receita sem comissão
- **Duplas:** suporte a pares de colaboradoras para um mesmo atendimento

---

## Design System

| Variável CSS | Valor |
|---|---|
| `--rosa` | `#c8697a` |
| `--rosa-dark` | `#a04f5e` |
| `--rosa-light` | `#f7e8eb` |
| `--dourado` | `#c9a96e` |
| `--verde` | `#4a9e7c` |
| `--cinza` | `#6b6b6b` |
| `--radius` | `12px` |
| `--sombra` | `0 2px 12px rgba(0,0,0,0.08)` |

Fontes: `Playfair Display` (títulos/logo) + `DM Sans` (corpo).
Layout mobile-first, max-width 480px, nav fixa no fundo.

---

## Changelog

### 2026-05-29
- Criação do arquivo `CONTEXT.md` com documentação completa do projeto
- **Fix:** algoritmo `distribuirAutomatico()` reformulado para o **Caso A** (sem duplas + todas as colaboradoras executam todos os serviços):
  - Antes: distribuía serviço a serviço com pontuação, onde o bônus `horaFixaAtivada` (+5.000.000) podia superar a penalidade de "outra collab já iniciou este cliente" (-500.000), dividindo um mesmo cliente entre colaboradoras
  - Agora: **Fase 1** — atribui CLIENTES (não serviços) às colaboradoras via round-robin por menor carga; **Fase 2** — agenda todos os serviços de cada cliente em sequência com a mesma colaboradora (sem interrupção)
  - Regras respeitadas: (a) igual número de clientes por colaboradora; (b) todos os serviços de um cliente ficam com a mesma colaboradora; (c) cliente em sequência MAKE→CABELO nunca é pausado por outra cliente com hora fixa; (d) clientes com hora fixa têm prioridade quando o horário chega, mas clientes sem hora fixa preenchem os intervalos se couberem inteiros antes do próximo hora fixa
  - Casos B/C (duplas ou restrição de serviço por colaboradora) mantêm o loop original inalterado
- **Fix 2 (refatoração completa):** algoritmo `distribuirAutomatico()` reescrito para suportar todos os cenários de evento:
  - **Unidade de distribuição**: igual número de **serviços executados** por colaboradora (não clientes)
  - **Decisão manter-junto vs dividir**: para cada cliente, simula as duas opções e escolhe a que produz menor diferença max−min de serviços entre collabs; se empate, prefere manter junto
  - **Collabs completas**: collab que executa todos os serviços de um cliente pode recebê-lo inteiro
  - **Divisão**: quando dividir equilibra melhor, cada serviço vai para a collab elegível de menor carga
  - **Ordem de serviços**: garantida via `cs.livre` (ex: MAKE sempre antes de CABELO — serviço N só começa após N-1 terminar, mesmo que sejam collabs diferentes)
  - **Hora_fixa**: respeitada no agendamento; clientes sem hora_fixa preenchem lacunas quando cabem inteiros antes do próximo horário fixo
  - **Detecção automática**: não depende de "sem restrição" — funciona com qualquer combinação de serviços por collab configurada nos checkboxes do evento
  - **Duplas**: pré-passo mantido; contagem de serviços já atribuídos pelas duplas é considerada no balanço dos demais clientes
- **Fix 3:** corrige caso simétrico (múltiplas collabs com mesma capacidade). Quando `colabsCompletas.length >= 2` (ex: DANI=MAKE+CABELO e PAULA=MAKE+CABELO), força "manter junto" sem comparar com dividir — a simulação de "dividir" produzia imbalance=0 no início e sempre vencia, dividindo clientes incorretamente entre collabs de mesma capacidade
