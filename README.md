# DashMais — Painel de Cobrança WhatsApp

O **DashMais** é um painel web para acompanhar os envios automáticos de cobrança por WhatsApp gerados a partir do fluxo n8n integrado ao BomControle, Evolution API e Google Sheets.

O objetivo é centralizar a visão de cobrança, permitindo acompanhar mensagens enviadas, clientes cobrados, inadimplência, pré-vencimentos, boletos emitidos e histórico de envios.

<img width="1852" height="925" alt="{21D00316-122F-4F3B-8EDA-8F1213E8AAA9}" src="https://github.com/user-attachments/assets/15214769-9a87-47f2-b97e-a93d15cc1437" />


---

## Visão geral

O fluxo atual funciona assim:

```mermaid
flowchart LR
  A[BomControle] --> B[n8n]
  B --> C[Buscar financeiro]
  C --> D[Filtrar boletos abertos com link]
  D --> E[Buscar cliente e contatos]
  E --> F[Selecionar telefone de cobrança]
  F --> G[Decidir tipo de envio]
  G --> H[Enviar WhatsApp via Evolution API]
  H --> I[Registrar envio]
  I --> J[Salvar log no Google Sheets]
  J --> K[DashMais]
```

O **n8n** roda uma vez por dia e decide automaticamente quem deve receber mensagem com base nas regras de cobrança.

O **Google Sheets** funciona como base de auditoria e histórico.

O **DashMais** lê esses dados e exibe indicadores visuais.

---

## Regras de envio

A automação envia mensagens em três situações principais:

| Tipo interno | Nome exibido | Quando envia |
|---|---|---|
| `emissao` | Boleto emitido | Quando o boleto foi criado/faturado no dia |
| `pre_vencimento` | Pré-vencimento | Um dia antes do vencimento |
| `vencido` | Inadimplência | Quando o boleto está vencido e respeita o intervalo configurado de cobrança |

Boletos quitados, sem link de boleto ou sem telefone de cobrança são ignorados.

---

## Fonte de dados

A base de dados do painel vem da planilha:

```text
Logs - Cobrança WhatsApp
```

As abas são mensais no formato:

```text
yyyy-MM
```

Exemplos:

```text
2026-05
2026-06
2026-07
```

Existe também uma aba:

```text
MODELO
```

Essa aba serve como modelo para criação automática das abas mensais.

---

## Colunas da planilha

A planilha deve conter as seguintes colunas:

| Coluna | Descrição |
|---|---|
| `Dia` | Data do envio no formato `dd/MM/yyyy` |
| `DataHora` | Data e hora do envio |
| `Cliente` | Nome do cliente |
| `Telefone` | Número usado no WhatsApp |
| `Valor` | Valor do boleto |
| `Vencimento` | Data de vencimento do boleto |
| `TipoEnvio` | Tipo do envio: `emissao`, `pre_vencimento` ou `vencido` |
| `Motivo` | Motivo legível do envio |
| `Status` | Status do envio, por exemplo `Enviado` |
| `IdBoleto` | ID da parcela/boleto no BomControle |
| `LinkBoleto` | Link do boleto |
| `Erro` | Erro do envio, quando houver |

---

## Indicadores do dashboard

O DashMais deve exibir:

### Cards principais

- Mensagens enviadas hoje
- Mensagens enviadas no mês
- Valor total cobrado no mês
- Quantidade de inadimplência
- Quantidade de pré-vencimentos
- Quantidade de boletos emitidos

### Gráficos

- Envios por dia
- Envios por tipo de envio
- Valor cobrado por tipo de envio
- Clientes mais cobrados por quantidade
- Clientes mais cobrados por valor

### Tabelas

- Últimos envios
- Histórico completo
- Clientes mais cobrados
- Inadimplências recentes

---

## Páginas do sistema

### Dashboard

Visão geral com cards, gráficos e últimos envios.

### Histórico de Envios

Tabela completa com filtros por:

- Mês
- Cliente
- Tipo de envio
- Status
- Busca por cliente ou telefone

### Clientes

Visão agrupada por cliente, mostrando:

- Total de mensagens enviadas
- Valor total cobrado
- Quantidade de inadimplências
- Último envio
- Telefone utilizado

---

## Stack recomendada

O projeto pode ser desenvolvido com:

- Next.js
- TypeScript
- Tailwind CSS
- shadcn/ui
- Recharts
- Google Sheets API
- Deploy na Vercel

---

## Variáveis de ambiente

Crie um arquivo `.env.local` baseado no exemplo abaixo:

```env
GOOGLE_SHEETS_SPREADSHEET_ID=ID_DA_PLANILHA
GOOGLE_SERVICE_ACCOUNT_EMAIL=email-da-service-account@projeto.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nSUA_CHAVE_PRIVADA_AQUI\n-----END PRIVATE KEY-----\n"
DASHMAIS_ACCESS_PASSWORD=sua_senha_de_acesso
```

> Importante: nunca envie o arquivo `.env.local` para o GitHub.

---

## Como obter as credenciais do Google

### 1. Criar uma Service Account

No Google Cloud:

```text
IAM e administrador → Contas de serviço → Criar conta de serviço
```

Nome sugerido:

```text
dashmais-sheets
```

### 2. Criar chave JSON

Dentro da Service Account:

```text
Chaves → Adicionar chave → Criar nova chave → JSON
```

O arquivo JSON baixado terá os campos:

```json
{
  "client_email": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
}
```

Use:

```text
client_email → GOOGLE_SERVICE_ACCOUNT_EMAIL
private_key → GOOGLE_PRIVATE_KEY
```

### 3. Compartilhar a planilha

Compartilhe a planilha `Logs - Cobrança WhatsApp` com o e-mail da Service Account.

Permissão mínima:

```text
Leitor
```

---

## Observação sobre chave privada

A variável `GOOGLE_PRIVATE_KEY` deve ficar em uma única linha no `.env`, mantendo os `\n`:

```env
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nMIIEv...\n-----END PRIVATE KEY-----\n"
```

No código, trate a chave assim:

```ts
const privateKey = process.env.GOOGLE_PRIVATE_KEY?.replace(/\\n/g, '\n');
```

---

## Segurança

O DashMais contém dados financeiros e links de boletos. Por isso:

- O painel deve ter tela de login com senha.
- A senha deve ser validada no servidor.
- Credenciais do Google não devem aparecer no frontend.
- O `.env.local` não deve ser versionado.
- O arquivo JSON da Service Account não deve ser enviado ao GitHub.
- Links de boleto devem abrir em nova aba com `rel="noopener noreferrer"`.

---

## Login

O acesso ao painel deve ser protegido pela variável:

```env
DASHMAIS_ACCESS_PASSWORD=sua_senha
```

Requisitos:

- Tela inicial antes do dashboard.
- Campo de senha.
- Mensagem discreta em caso de senha incorreta.
- Sessão simples via cookie ou localStorage.
- Botão de sair/logout.

---

## Responsividade

O painel deve funcionar bem em:

- Desktop
- Tablet
- Celular

Requisitos mobile:

- Menu lateral vira menu hambúrguer ou navegação inferior.
- Cards em uma coluna.
- Filtros empilhados.
- Tabelas com scroll horizontal ou cards compactos.
- Gráficos legíveis em telas pequenas.
- Nenhum elemento deve estourar a largura da tela.

---

## Normalização de dados

### Valor

A planilha pode trazer valores no formato:

```text
R$ 1.025,07
```

Para cálculos, converter para:

```text
1025.07
```

### TipoEnvio

Mapeamento visual:

| Valor na planilha | Texto no painel |
|---|---|
| `emissao` | Boleto emitido |
| `pre_vencimento` | Pré-vencimento |
| `vencido` | Inadimplência |

---

## Estados esperados

O sistema deve tratar:

- Carregando dados
- Erro ao carregar Google Sheets
- Nenhum dado encontrado para o mês
- Senha incorreta
- Dados demonstrativos
- Google Sheets conectado

---

## Deploy

### Vercel

1. Criar projeto na Vercel.
2. Configurar as variáveis de ambiente.
3. Fazer deploy.
4. Validar leitura da planilha.
5. Testar login e responsividade.

---

## Arquivos sensíveis

Não versionar:

```text
.env
.env.local
*.json de credencial Google
```

Adicionar ao `.gitignore`:

```gitignore
.env
.env.local
.env.*.local
google-service-account.json
```

---

## Evoluções futuras

Possíveis melhorias:

- Integração direta com SUMOS.
- Registro de erros em aba separada.
- Indicador de pagamento após cobrança.
- Ranking de clientes recorrentes em inadimplência.
- Alertas de falha no envio.
- Envio de relatório diário para equipe.
- Controle de mensagens por cliente.
- Histórico detalhado por boleto.
- Exportação CSV/PDF.
- Autenticação por usuário e senha individual.
- Integração com banco de dados próprio.

---

## Resumo

O DashMais é o painel de gestão dos envios de cobrança por WhatsApp da Suporte+.

Ele usa:

```text
n8n → Google Sheets → DashMais
```

A planilha funciona como base de auditoria, e o painel transforma os logs em indicadores úteis para acompanhamento financeiro e operacional.
