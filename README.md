# iClips Public API

Documentação de referência para integração com a iClips Public API.

**Base URL:** `https://public-api.iclips.com.br`

---

## Índice

- [Autenticação](#autenticação)
- [Formato das respostas](#formato-das-respostas)
- [Erros](#erros)
- [Endpoints](#endpoints)
  - **Financeiro**
  - [GET /api/v1/financial/accounts — Listar contas bancárias](#1-get-apiv1financialaccounts)
  - [GET /api/v1/financial/entries — Listar lançamentos](#2-get-apiv1financialentries)
  - [GET /api/v1/financial/entries/{id} — Detalhe do lançamento](#3-get-apiv1financialentriesid)
  - [GET /api/v1/financial/invoices — Listar notas fiscais](#4-get-apiv1financialinvoices)
  - [GET /api/v1/financial/invoices/{id} — Detalhe da nota fiscal](#5-get-apiv1financialinvoicesid)
  - **Projetos e Peças**
  - [POST /api/v1/jobs — Criar projeto (Job)](#6-post-apiv1jobs)
  - [PUT /api/v1/jobs/{id} — Atualizar projeto (Job)](#7-put-apiv1jobsid)
  - [DELETE /api/v1/jobs/{id} — Excluir projeto (Job)](#8-delete-apiv1jobsid)
  - [POST /api/v1/jobs/{id}/pecas — Vincular peça a um projeto](#9-post-apiv1jobsidpecas)
  - [PUT /api/v1/jobs/{id}/pecas/{jobPecaId} — Atualizar peça de um projeto](#10-put-apiv1jobsidpecasjobpecaid)
  - [POST /api/v1/jobs/{id}/tarefas — Criar tarefa em um projeto](#11-post-apiv1jobsidtarefas)
  - [PUT /api/v1/jobs/{id}/tarefas/{tarefaId} — Atualizar tarefa de um projeto](#12-put-apiv1jobsidtarefastarefaid)
  - [GET /api/v1/pieces — Listar peças](#13-get-apiv1pieces)
  - [GET /api/v1/pieces/{id} — Detalhe da peça](#14-get-apiv1piecesid)
  - [POST /api/v1/pieces — Criar peça](#15-post-apiv1pieces)
  - [PUT /api/v1/pieces/{id} — Atualizar peça](#16-put-apiv1piecesid)
  - [PATCH /api/v1/pieces/{id}/checklist — Marcar item do checklist](#17-patch-apiv1piecesidchecklist)
  - [DELETE /api/v1/pieces/{id} — Excluir peça](#18-delete-apiv1piecesid)
  - [GET /api/v1/workflow-templates — Listar workflow templates](#19-get-apiv1workflow-templates)
  - [GET /api/v1/workflow-templates/{id} — Detalhe do workflow template](#20-get-apiv1workflow-templatesid)
  - [GET /api/v1/piece-categories — Listar categorias de peça](#21-get-apiv1piece-categories)
  - [GET /api/v1/piece-categories/dropdown — Categorias para dropdown](#22-get-apiv1piece-categoriesdropdown)
  - **Extração de Dados**
  - [GET /api/v1/projetos — Extrair projetos com hierarquia completa](#23-get-apiv1projetos)
- [Paginação](#paginação)
- [Limites e boas práticas](#limites-e-boas-práticas)

---

## Autenticação

Todos os endpoints requerem autenticação via **API Key** no cabeçalho HTTP.

| Header      | Obrigatório | Descrição                          |
|-------------|-------------|------------------------------------|
| `X-Api-Key` | Sim         | Chave de API fornecida pela iClips |

A API Key é um token gerado pela plataforma iClips para cada agência. Ela deve ser tratada como credencial sigilosa e **nunca exposta em código front-end ou repositórios públicos**.

**Exemplo:**
```
X-Api-Key: eyJwYXlsb2FkIjoieyJBZ2VuY3lJZCI6MTIzfSIsInNpZ25hdHVyZSI6Ii4uLiJ9
```

### Respostas de erro de autenticação

Requisições sem o header ou com chave inválida retornam **HTTP 401** com o seguinte formato:

```json
{
  "error": "API Key obrigatória",
  "message": "O header 'X-Api-Key' é obrigatório para acessar este recurso",
  "traceId": "0HNBD52H7RFAGB:00000001",
  "timestamp": "2025-02-10T13:00:00"
}
```

| Situação                              | HTTP | Campo `error`           |
|---------------------------------------|------|-------------------------|
| Header `X-Api-Key` ausente            | 401  | `API Key obrigatória`   |
| Chave inválida, malformada ou expirada | 401  | `API Key inválida`      |

---

## Formato das respostas

### Resposta paginada

Endpoints de listagem retornam o seguinte envelope:

```json
{
  "data": [ ... ],
  "meta": {
    "total": 150,
    "limit": 50,
    "offset": 0,
    "hasMore": true,
    "from": "2025-01-01",
    "to": "2025-03-31"
  }
}
```

| Campo          | Tipo    | Descrição                                                |
|----------------|---------|----------------------------------------------------------|
| `data`         | array   | Lista de objetos retornados                              |
| `meta.total`   | integer | Total de registros existentes no período filtrado        |
| `meta.limit`   | integer | Quantidade de registros por página                       |
| `meta.offset`  | integer | Deslocamento atual da paginação                          |
| `meta.hasMore` | boolean | Indica se existem mais páginas                           |
| `meta.from`    | string  | Data de início do filtro aplicado (`YYYY-MM-DD`)         |
| `meta.to`      | string  | Data de fim do filtro aplicado (`YYYY-MM-DD`)            |

### Resposta de detalhe (por ID)

```json
{
  "data": { ... }
}
```

---

## Erros

Existem dois formatos de erro, dependendo da origem:

### Erros de autenticação — HTTP 401

Retornados pelo gateway antes de chegar nos endpoints.

```json
{
  "error": "API Key inválida",
  "message": "A API Key fornecida é inválida, malformada ou expirada",
  "traceId": "0HNBD52H7RFAGB:00000001",
  "timestamp": "2025-02-10T13:00:00"
}
```

### Erros de validação e negócio — HTTP 400 / 404

Retornados pelos endpoints de dados.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "O parâmetro 'from' é obrigatório.",
    "details": [
      { "field": "from", "issue": "missing" }
    ]
  }
}
```

| Código             | HTTP | Quando ocorre                                                       |
|--------------------|------|---------------------------------------------------------------------|
| `VALIDATION_ERROR` | 400  | Parâmetro ausente, inválido, fora do range ou período acima de 366 dias |
| `NOT_FOUND`        | 404  | Recurso não encontrado ou não pertence à agência autenticada        |

**Exemplos de `details` para erros de validação:**

| `field`       | `issue`         | Motivo                                          |
|---------------|-----------------|-------------------------------------------------|
| `from`        | `missing`       | Parâmetro `from` não enviado                    |
| `to`          | `missing`       | Parâmetro `to` não enviado                      |
| `limit`       | `out_of_range`  | `limit` fora do intervalo 1–200                 |
| `status`      | `invalid`       | Valor de `status` não reconhecido               |
| `date_range`  | `exceeded`      | Diferença entre `from` e `to` maior que 366 dias |

---

## Endpoints

### 1. GET /api/v1/financial/accounts

Retorna todas as contas bancárias ativas da agência. Não possui filtros nem paginação.

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/financial/accounts" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{
  "data": [
    {
      "id": 12,
      "name": "Conta Principal Bradesco",
      "status": "active",
      "isDefault": true,
      "openingDate": "2020-03-15T00:00:00",
      "bank": {
        "number": "237",
        "agency": "1234-5",
        "accountNumber": "00012345-6"
      }
    },
    {
      "id": 17,
      "name": "Caixa Interno",
      "status": "active",
      "isDefault": false,
      "openingDate": null,
      "bank": {
        "number": null,
        "agency": null,
        "accountNumber": null
      }
    }
  ]
}
```

#### Campos da resposta

| Campo                | Tipo            | Descrição                                         |
|----------------------|-----------------|---------------------------------------------------|
| `id`                 | integer         | Identificador único da conta                      |
| `name`               | string / null   | Nome descritivo da conta                          |
| `status`             | string          | Status da conta. Valor fixo: `active`             |
| `isDefault`          | boolean         | Indica se é a conta padrão da agência             |
| `openingDate`        | datetime / null | Data de abertura da conta                         |
| `bank.number`        | string / null   | Código do banco (ex: `237` para Bradesco)         |
| `bank.agency`        | string / null   | Número da agência bancária                        |
| `bank.accountNumber` | string / null   | Número da conta bancária                          |

---

### 2. GET /api/v1/financial/entries

Lista os lançamentos financeiros da agência com suporte a filtros e paginação.

#### Parâmetros de filtro (query string)

| Parâmetro        | Tipo    | Obrigatório | Descrição                                                                                     |
|------------------|---------|-------------|-----------------------------------------------------------------------------------------------|
| `from`           | date    | **Sim**     | Data de início do período (`YYYY-MM-DD`). Filtra pelo campo `entryDate`.                      |
| `to`             | date    | **Sim**     | Data de fim do período (`YYYY-MM-DD`). Máximo de 366 dias de diferença em relação a `from`.   |
| `type`           | string  | Não         | Tipo do lançamento. Valores aceitos: `Entrada`, `Saída`, `A Receber`, `A Pagar`               |
| `account_id`     | integer | Não         | Filtra pelo ID da conta bancária (obtido via `/accounts`)                                     |
| `category_id`    | integer | Não         | Filtra pelo ID da categoria do lançamento                                                     |
| `dre_indicator`  | string  | Não         | Filtra pelo indicador DRE da categoria raiz. Ver valores válidos abaixo.                      |
| `destination_id` | integer | Não         | Filtra pelo ID do destinatário/fornecedor do lançamento                                       |
| `job_id`         | integer | Não         | Filtra pelo ID do job vinculado ao lançamento                                                 |
| `limit`          | integer | Não         | Registros por página. Padrão: `50`. Mín: `1`. Máx: `200`.                                    |
| `offset`         | integer | Não         | Deslocamento para paginação. Padrão: `0`.                                                     |

**Valores válidos para `dre_indicator`:**

| Valor                                  | Descrição                            |
|----------------------------------------|--------------------------------------|
| `DESPESAS OPERACIONAIS`                | Despesas operacionais gerais         |
| `DESPESAS OPERACIONAIS CUSTO VARIÁVEL` | Custo variável                       |
| `DESPESAS OPERACIONAIS CUSTO FIXO`     | Custo fixo                           |
| `DESPESAS NÃO OPERACIONAIS`            | Despesas fora do escopo operacional  |
| `IMPOSTOS`                             | Lançamentos de impostos              |
| `PARTICIPAÇÃO NOS LUCROS`              | PLR e similares                      |
| `RECEITAS OPERACIONAIS`                | Faturamento principal                |
| `RECEITAS NÃO OPERACIONAIS`            | Outras receitas                      |
| `RECEITAS TOTAIS`                      | Totalização de receitas              |
| `NENHUM`                               | Sem indicador DRE definido           |
| `OUTRO`                                | Demais categorias                    |

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/financial/entries?from=2025-01-01&to=2025-03-31&type=Sa%C3%ADda&limit=20&offset=0" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{
  "data": [
    {
      "id": 4801,
      "parentEntryId": 0,
      "jobId": 230,
      "title": "Pagamento fornecedor gráfica",
      "description": "Referente ao job 230 - Campanha Verão 2025",
      "type": "Saída",
      "entryDate": "2025-02-10T00:00:00",
      "dueDate": "2025-02-10T00:00:00",
      "competenceDate": "2025-02-01T00:00:00",
      "amount": 3500.00,
      "discount": null,
      "interest": null,
      "fine": null,
      "otherRetentions": null,
      "irrf": 52.50,
      "issrf": null,
      "taxPercentage": 1.5,
      "category": {
        "id": 18,
        "name": "Produção Gráfica",
        "parentName": "Fornecedores",
        "dreIndicator": "DESPESAS OPERACIONAIS"
      },
      "account": {
        "id": 12,
        "name": "Conta Principal Bradesco",
        "bank": "Bradesco",
        "agency": "1234-5",
        "accountNumber": "00012345-6",
        "companyName": "Agência XYZ Comunicação Ltda",
        "cnpj": "12.345.678/0001-90"
      },
      "invoice": {
        "number": null,
        "issuanceDate": null,
        "description": null
      },
      "destination": {
        "id": 95,
        "name": "Gráfica Expressão Ltda",
        "document": "98.765.432/0001-11"
      },
      "costCenters": [
        {
          "id": 3,
          "name": "Atendimento",
          "amount": 3500.00,
          "percentage": 100.00
        }
      ]
    }
  ],
  "meta": {
    "total": 87,
    "limit": 20,
    "offset": 0,
    "hasMore": true,
    "from": "2025-01-01",
    "to": "2025-03-31"
  }
}
```

#### Campos da resposta

| Campo                      | Tipo            | Descrição                                                                       |
|----------------------------|-----------------|---------------------------------------------------------------------------------|
| `id`                       | integer         | Identificador único do lançamento                                               |
| `parentEntryId`            | integer         | ID do lançamento pai (parcelamentos). `0` se não houver                         |
| `jobId`                    | integer         | ID do job vinculado. `0` se não vinculado                                       |
| `title`                    | string / null   | Título do lançamento                                                            |
| `description`              | string / null   | Descrição detalhada                                                             |
| `type`                     | string          | Tipo: `Entrada`, `Saída`, `A Receber`, `A Pagar`                                |
| `entryDate`                | datetime / null | Data de realização do lançamento                                                |
| `dueDate`                  | datetime / null | Data de vencimento/pagamento                                                    |
| `competenceDate`           | datetime / null | Data de competência contábil                                                    |
| `amount`                   | decimal         | Valor bruto do lançamento                                                       |
| `discount`                 | decimal / null  | Desconto aplicado                                                               |
| `interest`                 | decimal / null  | Juros incidentes                                                                |
| `fine`                     | decimal / null  | Multa incidente                                                                 |
| `otherRetentions`          | decimal / null  | Outras retenções                                                                |
| `irrf`                     | decimal / null  | Valor do IRRF retido                                                            |
| `issrf`                    | decimal / null  | Valor do ISSRF retido                                                           |
| `taxPercentage`            | decimal / null  | Percentual de imposto                                                           |
| `category.id`              | integer         | ID da categoria do lançamento                                                   |
| `category.name`            | string / null   | Nome da categoria                                                               |
| `category.parentName`      | string / null   | Nome da categoria pai                                                           |
| `category.dreIndicator`    | string          | Indicador DRE da categoria raiz                                                 |
| `account.id`               | integer         | ID da conta bancária                                                            |
| `account.name`             | string / null   | Nome da conta                                                                   |
| `account.bank`             | string / null   | Nome do banco                                                                   |
| `account.agency`           | string / null   | Agência bancária                                                                |
| `account.accountNumber`    | string / null   | Número da conta bancária                                                        |
| `account.companyName`      | string / null   | Razão social da empresa titular da conta                                        |
| `account.cnpj`             | string / null   | CNPJ da empresa titular da conta                                                |
| `invoice.number`           | string / null   | Número da nota fiscal vinculada ao lançamento                                  |
| `invoice.issuanceDate`     | datetime / null | Data de emissão da nota fiscal vinculada                                        |
| `invoice.description`      | string / null   | Discriminação da nota fiscal vinculada                                          |
| `destination.id`           | integer         | ID do destinatário (cliente ou fornecedor)                                      |
| `destination.name`         | string / null   | Nome do destinatário                                                            |
| `destination.document`     | string / null   | CPF ou CNPJ do destinatário                                                     |
| `costCenters`              | array           | Lista de centros de custo rateados no lançamento (pode ser vazia)               |
| `costCenters[].id`         | integer         | ID do centro de custo                                                           |
| `costCenters[].name`       | string          | Nome do centro de custo                                                         |
| `costCenters[].amount`     | decimal         | Valor rateado para este centro de custo                                         |
| `costCenters[].percentage` | decimal         | Percentual do rateio (0–100)                                                    |

#### Resposta — 400 Bad Request

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "O período máximo de consulta é de 366 dias.",
    "details": [
      { "field": "date_range", "issue": "exceeded" }
    ]
  }
}
```

---

### 3. GET /api/v1/financial/entries/{id}

Retorna o detalhe de um lançamento financeiro específico.

#### Parâmetros de rota

| Parâmetro | Tipo    | Obrigatório | Descrição                   |
|-----------|---------|-------------|-----------------------------|
| `id`      | integer | **Sim**     | Identificador do lançamento |

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/financial/entries/4801" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{
  "data": {
    "id": 4801,
    "parentEntryId": 0,
    "jobId": 230,
    "title": "Pagamento fornecedor gráfica",
    "description": "Referente ao job 230 - Campanha Verão 2025",
    "type": "Saída",
    "entryDate": "2025-02-10T00:00:00",
    "dueDate": "2025-02-10T00:00:00",
    "competenceDate": "2025-02-01T00:00:00",
    "amount": 3500.00,
    "discount": null,
    "interest": null,
    "fine": null,
    "otherRetentions": null,
    "irrf": 52.50,
    "issrf": null,
    "taxPercentage": 1.5,
    "category": {
      "id": 18,
      "name": "Produção Gráfica",
      "parentName": "Fornecedores",
      "dreIndicator": "DESPESAS OPERACIONAIS"
    },
    "account": {
      "id": 12,
      "name": "Conta Principal Bradesco",
      "bank": "Bradesco",
      "agency": "1234-5",
      "accountNumber": "00012345-6",
      "companyName": "Agência XYZ Comunicação Ltda",
      "cnpj": "12.345.678/0001-90"
    },
    "invoice": {
      "number": null,
      "issuanceDate": null,
      "description": null
    },
    "destination": {
      "id": 95,
      "name": "Gráfica Expressão Ltda",
      "document": "98.765.432/0001-11"
    },
    "costCenters": [
      {
        "id": 3,
        "name": "Atendimento",
        "amount": 3500.00,
        "percentage": 100.00
      }
    ]
  }
}
```

> Os campos da resposta são idênticos aos descritos na [tabela do endpoint de listagem](#campos-da-resposta-1).

#### Resposta — 404 Not Found

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Recurso não encontrado."
  }
}
```

---

### 4. GET /api/v1/financial/invoices

Lista as notas fiscais emitidas pela agência com suporte a filtros e paginação.

#### Parâmetros de filtro (query string)

| Parâmetro | Tipo    | Obrigatório | Descrição                                                                                    |
|-----------|---------|-------------|----------------------------------------------------------------------------------------------|
| `from`    | date    | **Sim**     | Data de início do período (`YYYY-MM-DD`). Filtra pela data de emissão da nota.               |
| `to`      | date    | **Sim**     | Data de fim do período (`YYYY-MM-DD`). Máximo de 366 dias de diferença em relação a `from`.  |
| `status`  | string  | Não         | Status da nota fiscal. Valores aceitos: `pending`, `issued`, `cancelled`, `error`            |
| `limit`   | integer | Não         | Registros por página. Padrão: `50`. Mín: `1`. Máx: `200`.                                   |
| `offset`  | integer | Não         | Deslocamento para paginação. Padrão: `0`.                                                    |

**Valores para `status`:**

| Valor       | Descrição                                 |
|-------------|-------------------------------------------|
| `pending`   | Nota ainda não emitida / aguardando envio |
| `issued`    | Nota emitida com sucesso                  |
| `cancelled` | Nota cancelada                            |
| `error`     | Falha no processo de emissão              |

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/financial/invoices?from=2025-01-01&to=2025-03-31&status=issued&limit=10&offset=0" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{
  "data": [
    {
      "id": 310,
      "number": "000123",
      "verificationCode": "ABC12345",
      "status": "issued",
      "issuanceDate": "2025-02-14T10:32:00",
      "competenceDate": "2025-02-01T00:00:00",
      "serviceAmount": 15000.00,
      "deductionsAmount": 0.00,
      "netAmount": 15000.00,
      "taxBaseAmount": 15000.00,
      "issRate": 5.00,
      "issAmount": 750.00,
      "issRetained": false,
      "taxes": {
        "pis": 97.50,
        "cofins": 450.00,
        "inss": null,
        "ir": 225.00,
        "csll": 135.00,
        "issRetainedAmount": null,
        "otherRetentions": null
      },
      "discrimination": "Prestação de serviços de comunicação e marketing referente ao mês de fevereiro de 2025.",
      "client": {
        "name": "Empresa Contratante S/A",
        "document": "11.222.333/0001-44",
        "municipalRegistration": "123456",
        "email": "financeiro@contratante.com.br",
        "city": "São Paulo",
        "state": "SP"
      },
      "issuer": {
        "name": "Agência XYZ Comunicação Ltda",
        "tradeName": "Agência XYZ",
        "document": "12.345.678/0001-90",
        "municipalRegistration": "654321"
      },
      "rps": {
        "id": "890",
        "series": "A",
        "batchId": "45"
      },
      "cancellation": null
    }
  ],
  "meta": {
    "total": 34,
    "limit": 10,
    "offset": 0,
    "hasMore": true,
    "from": "2025-01-01",
    "to": "2025-03-31"
  }
}
```

#### Campos da resposta

| Campo                          | Tipo            | Descrição                                                          |
|--------------------------------|-----------------|--------------------------------------------------------------------|
| `id`                           | integer         | Identificador único da nota fiscal                                 |
| `number`                       | string / null   | Número da nota fiscal emitida pelo município                       |
| `verificationCode`             | string / null   | Código de verificação da NFS-e                                     |
| `status`                       | string          | Status: `pending`, `issued`, `cancelled`, `error`                 |
| `issuanceDate`                 | datetime / null | Data e hora de emissão                                             |
| `competenceDate`               | datetime / null | Data de competência do serviço                                     |
| `serviceAmount`                | decimal         | Valor bruto dos serviços prestados                                 |
| `deductionsAmount`             | decimal / null  | Total de deduções aplicadas                                        |
| `netAmount`                    | decimal / null  | Valor líquido (serviceAmount − deductionsAmount)                   |
| `taxBaseAmount`                | decimal / null  | Base de cálculo dos tributos                                       |
| `issRate`                      | decimal / null  | Alíquota do ISS (em percentual)                                    |
| `issAmount`                    | decimal / null  | Valor do ISS calculado                                             |
| `issRetained`                  | boolean         | `true` se o ISS será retido na fonte pelo tomador                  |
| `taxes.pis`                    | decimal / null  | Valor do PIS retido                                                |
| `taxes.cofins`                 | decimal / null  | Valor do COFINS retido                                             |
| `taxes.inss`                   | decimal / null  | Valor do INSS retido                                               |
| `taxes.ir`                     | decimal / null  | Valor do IR retido                                                 |
| `taxes.csll`                   | decimal / null  | Valor do CSLL retido                                               |
| `taxes.issRetainedAmount`      | decimal / null  | Valor do ISS retido na fonte (quando `issRetained` = `true`)       |
| `taxes.otherRetentions`        | decimal / null  | Outras retenções                                                   |
| `discrimination`               | string / null   | Discriminação dos serviços conforme consta na nota fiscal          |
| `client.name`                  | string / null   | Nome/razão social do tomador de serviço                            |
| `client.document`              | string / null   | CPF ou CNPJ do tomador                                             |
| `client.municipalRegistration` | string / null   | Inscrição municipal do tomador                                     |
| `client.email`                 | string / null   | E-mail do tomador                                                  |
| `client.city`                  | string / null   | Cidade do tomador                                                  |
| `client.state`                 | string / null   | Estado (UF) do tomador                                             |
| `issuer.name`                  | string / null   | Razão social do prestador (emitente)                               |
| `issuer.tradeName`             | string / null   | Nome fantasia do prestador                                         |
| `issuer.document`              | string / null   | CNPJ do prestador                                                  |
| `issuer.municipalRegistration` | string / null   | Inscrição municipal do prestador                                   |
| `rps.id`                       | string / null   | Número do RPS (Recibo Provisório de Serviços)                      |
| `rps.series`                   | string / null   | Série do RPS                                                       |
| `rps.batchId`                  | string / null   | Identificador do lote de envio do RPS                              |
| `cancellation`                 | object / null   | Presente apenas quando `status` = `cancelled`                      |
| `cancellation.reasonCode`      | integer         | Código do motivo do cancelamento                                   |
| `cancellation.description`     | string / null   | Descrição do motivo do cancelamento                                |

#### Resposta — 400 Bad Request

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Valores válidos: pending, issued, cancelled, error.",
    "details": [
      { "field": "status", "issue": "invalid" }
    ]
  }
}
```

---

### 5. GET /api/v1/financial/invoices/{id}

Retorna o detalhe de uma nota fiscal específica.

#### Parâmetros de rota

| Parâmetro | Tipo    | Obrigatório | Descrição                    |
|-----------|---------|-------------|------------------------------|
| `id`      | integer | **Sim**     | Identificador da nota fiscal |

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/financial/invoices/310" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{
  "data": {
    "id": 310,
    "number": "000123",
    "verificationCode": "ABC12345",
    "status": "issued",
    "issuanceDate": "2025-02-14T10:32:00",
    "competenceDate": "2025-02-01T00:00:00",
    "serviceAmount": 15000.00,
    "deductionsAmount": 0.00,
    "netAmount": 15000.00,
    "taxBaseAmount": 15000.00,
    "issRate": 5.00,
    "issAmount": 750.00,
    "issRetained": false,
    "taxes": {
      "pis": 97.50,
      "cofins": 450.00,
      "inss": null,
      "ir": 225.00,
      "csll": 135.00,
      "issRetainedAmount": null,
      "otherRetentions": null
    },
    "discrimination": "Prestação de serviços de comunicação e marketing referente ao mês de fevereiro de 2025.",
    "client": {
      "name": "Empresa Contratante S/A",
      "document": "11.222.333/0001-44",
      "municipalRegistration": "123456",
      "email": "financeiro@contratante.com.br",
      "city": "São Paulo",
      "state": "SP"
    },
    "issuer": {
      "name": "Agência XYZ Comunicação Ltda",
      "tradeName": "Agência XYZ",
      "document": "12.345.678/0001-90",
      "municipalRegistration": "654321"
    },
    "rps": {
      "id": "890",
      "series": "A",
      "batchId": "45"
    },
    "cancellation": null
  }
}
```

> Os campos da resposta são idênticos aos descritos na [tabela do endpoint de listagem](#campos-da-resposta-3).

#### Resposta — 404 Not Found

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Recurso não encontrado."
  }
}
```

---

### 6. POST /api/v1/jobs

Cria um novo Job (Projeto) no iClips.

#### Headers

| Header         | Obrigatório | Descrição          |
|----------------|-------------|--------------------|
| `X-Api-Key`    | Sim         | Chave da agência   |
| `Content-Type` | Sim         | `application/json` |

#### Exemplo de requisição

```bash
curl -X POST "https://public-api.iclips.com.br/api/v1/jobs" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Campanha Verão 2025",
    "clienteId": 42,
    "status": 6,
    "entradaDate": "2025-06-01",
    "conclusaoDate": "2025-07-31",
    "tarefasInput": [
      {
        "titulo": "Reunião de briefing",
        "descricao": "Alinhar detalhes com o cliente",
        "inicioDate": "2025-06-02",
        "fimDate": "2025-06-02",
        "prioridade": 1,
        "ordemExibicao": 1
      }
    ]
  }'
```

#### Campos do Request Body

| Campo            | Tipo           | Obrigatório | Descrição                                                               |
|------------------|----------------|-------------|-------------------------------------------------------------------------|
| `titulo`         | string         | **Sim**     | Título do job. Máx 100 caracteres.                                      |
| `clienteId`      | integer / null | Cond.*      | ID do cliente. Mutuamente exclusivo com `grupoClienteId`.               |
| `grupoClienteId` | integer / null | Cond.*      | ID do grupo de cliente. Mutuamente exclusivo com `clienteId`.           |
| `entradaDate`    | date / null    | Não         | Data de entrada (`YYYY-MM-DD`). Deve ser ≤ `conclusaoDate`.             |
| `aprovacaoDate`  | date / null    | Não         | Data de aprovação (`YYYY-MM-DD`).                                       |
| `conclusaoDate`  | date / null    | Não         | Data de conclusão (`YYYY-MM-DD`).                                       |
| `status`         | integer        | **Sim**     | Código do status (ver tabela abaixo).                                   |
| `prioridade`     | integer / null | Não         | 1=Baixa, 2=Média, 3=Alta.                                               |
| `servico`        | string / null  | Não         | Nome do serviço.                                                        |
| `campanha`       | string / null  | Não         | Nome da campanha vinculada.                                             |
| `descricao`      | string / null  | Não         | Descrição do job.                                                       |
| `briefing`       | string / null  | Não         | Texto de briefing.                                                      |
| `verba`          | decimal / null | Não         | Verba disponível.                                                       |
| `idVerba`        | integer / null | Não         | ID da verba vinculada.                                                  |
| `idFeeMensal`    | integer / null | Não         | ID do fee mensal vinculado.                                             |
| `funcionarioId`  | integer / null | Não         | ID do funcionário responsável. Se omitido, usa o usuário resolvido pela API Key. |
| `auxiliarId`     | integer / null | Não         | ID do funcionário auxiliar.                                             |
| `modeloIds`      | integer[]      | Não         | IDs dos modelos de job para geração automática de peças/tarefas.        |
| `pecaIds`        | integer[]      | Não         | IDs de peças adicionais.                                                |
| `tarefasInput`   | object[]       | Não         | Tarefas manuais adicionais (ver campos abaixo).                         |

*`clienteId` e `grupoClienteId` são mutuamente exclusivos; exatamente um deve ser informado.*

#### Campos de `tarefasInput[]`

⚠️ Estes nomes são diferentes dos campos retornados por `GET /api/v1/projetos` (que usa
`tituloAtividade`, `tempoEstimado`, `inicioPlanejado`, `fimPlanejado`) — aquele é o schema de
**leitura**; o schema de **escrita** (criação) é este abaixo.

| Campo           | Tipo           | Obrigatório | Descrição                                    |
|-----------------|----------------|-------------|-----------------------------------------------|
| `titulo`        | string         | **Sim**     | Título da tarefa. Máx 150 caracteres.          |
| `descricao`     | string / null  | Não         | Descrição da tarefa.                           |
| `inicioDate`    | date / null    | Não         | Data de início da tarefa (`YYYY-MM-DD`).       |
| `fimDate`       | date / null    | Não         | Data de fim da tarefa (`YYYY-MM-DD`).          |
| `prioridade`    | integer / null | Não         | 1=Baixa, 2=Média, 3=Alta.                      |
| `ordemExibicao` | integer / null | Não         | Ordem de exibição da tarefa na lista.          |

#### Valores de `status`

| Código | Nome                 |
|--------|----------------------|
| `1`    | Proposta Comercial   |
| `2`    | Concorrência         |
| `3`    | Risco                |
| `5`    | Aguardando Aprovação |
| `6`    | Autorizado           |
| `7`    | Finalizado           |
| `8`    | Cancelado            |
| `9`    | Aberto pelo Cliente  |
| `10`   | Pré Produção         |
| `11`   | Stand By             |

#### Resposta — 201 Created

```json
{
  "jobId": 1234,
  "titulo": "Campanha Verão 2025",
  "sigla": "JOB-1234",
  "status": 6,
  "totalPecas": 0,
  "totalTarefas": 0
}
```

| Campo          | Tipo    | Descrição                              |
|----------------|---------|----------------------------------------|
| `jobId`        | integer | ID do job criado                       |
| `titulo`       | string  | Título do job                          |
| `sigla`        | string  | Sigla gerada automaticamente           |
| `status`       | integer | Código do status                       |
| `totalPecas`   | integer | Total de peças criadas                 |
| `totalTarefas` | integer | Total de tarefas criadas               |

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `titulo` vazio, `status` inválido, `clienteId` e `grupoClienteId` simultâneos, item de `tarefasInput[]` sem `titulo` |
| 401  | API Key ausente ou inválida |

---

### 7. PUT /api/v1/jobs/{id}

Atualiza um Job (Projeto) existente no iClips.

#### Parâmetros de rota

| Parâmetro | Tipo    | Obrigatório | Descrição            |
|-----------|---------|-------------|----------------------|
| `id`      | integer | **Sim**     | Identificador do job |

#### Exemplo de requisição

```bash
curl -X PUT "https://public-api.iclips.com.br/api/v1/jobs/1234" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Campanha Verão 2025 — Revisado",
    "clienteId": 42,
    "status": 7
  }'
```

#### Campos do Request Body

Mesmos campos de metadado do POST: `titulo`, `clienteId`/`grupoClienteId`, `entradaDate`,
`aprovacaoDate`, `conclusaoDate`, `status`, `prioridade`, `servico`, `campanha`, `descricao`,
`briefing`, `verba`, `idVerba`, `idFeeMensal`, `funcionarioId`, `auxiliarId`.

| Campo            | Tipo           | Obrigatório | Descrição                                           |
|------------------|----------------|-------------|-----------------------------------------------------|
| `titulo`         | string         | **Sim**     | Título do job. Máx 100 caracteres.                  |
| `status`         | integer        | **Sim**     | Código do status (mesmos valores do POST).          |
| `funcionarioId`  | integer / null | Não         | ID do funcionário responsável.                      |
| `auxiliarId`     | integer / null | Não         | ID do funcionário auxiliar.                         |

⚠️ **`modeloIds`, `pecaIds` e `tarefasInput` não são suportados neste endpoint.** Se enviados,
são ignorados silenciosamente — a lista de peças e de tarefas do job não é alterada pelo PUT
nesta versão da API.

#### Resposta — 200 OK

```json
{
  "jobId": 1234,
  "titulo": "Campanha Verão 2025 — Revisado",
  "sigla": "JOB-1234",
  "status": 7,
  "totalPecas": 0,
  "totalTarefas": 0
}
```

Mesmo formato de resposta do POST (ver tabela de campos na seção 6). `totalPecas` e
`totalTarefas` refletem o que já existia no job antes do update — não são recalculados por
este endpoint.

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `titulo` vazio, `status` inválido, datas inconsistentes, `clienteId` e `grupoClienteId` simultâneos |
| 401  | API Key ausente ou inválida |
| 404  | Job não encontrado |

---

### 8. DELETE /api/v1/jobs/{id}

Exclui um Job (Projeto) do iClips.

#### Parâmetros de rota

| Parâmetro | Tipo    | Obrigatório | Descrição            |
|-----------|---------|-------------|----------------------|
| `id`      | integer | **Sim**     | Identificador do job |

#### Exemplo de requisição

```bash
curl -X DELETE "https://public-api.iclips.com.br/api/v1/jobs/1234" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{ "success": true }
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 401  | API Key ausente ou inválida |
| 404  | Job não encontrado |

---

### 9. POST /api/v1/jobs/{id}/pecas

Vincula uma peça do catálogo a um Job (Projeto) já existente.

#### Parâmetros de rota

| Parâmetro | Tipo    | Obrigatório | Descrição            |
|-----------|---------|-------------|----------------------|
| `id`      | integer | **Sim**     | Identificador do job |

#### Exemplo de requisição

```bash
curl -X POST "https://public-api.iclips.com.br/api/v1/jobs/1234/pecas" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "pecaId": 5,
    "titulo": "Banner Instagram",
    "prioridade": 2
  }'
```

#### Campos do Request Body

| Campo        | Tipo           | Obrigatório | Descrição                                                      |
|--------------|----------------|-------------|------------------------------------------------------------------|
| `pecaId`     | integer        | **Sim**     | ID da peça no catálogo (ver `GET /api/v1/pieces`).                |
| `titulo`     | string / null  | Não         | Título da peça no job. Se omitido, usa o nome da peça do catálogo. |
| `descricao`  | string / null  | Não         | Descrição da peça no job. Se omitido, usa a instrução do catálogo. |
| `prioridade` | integer / null | Não         | 1=Baixa, 2=Média, 3=Alta. Padrão: Média.                          |

#### Resposta — 201 Created

```json
{
  "jobId": 1234,
  "jobPecaId": 42,
  "pecaId": 5,
  "titulo": "Banner Instagram",
  "status": 1
}
```

| Campo       | Tipo    | Descrição                                                        |
|-------------|---------|-------------------------------------------------------------------|
| `jobId`     | integer | ID do job                                                          |
| `jobPecaId` | integer | ID da peça vinculada ao job — **diferente** do `pecaId` do catálogo, use este id no `PUT` |
| `pecaId`    | integer | ID da peça no catálogo                                             |
| `titulo`    | string  | Título da peça no job                                              |
| `status`    | integer | Código de status da peça no job                                    |

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `pecaId` ausente, peça já vinculada a este job |
| 401  | API Key ausente ou inválida |
| 404  | Job não encontrado, peça não encontrada no catálogo |

---

### 10. PUT /api/v1/jobs/{id}/pecas/{jobPecaId}

Atualiza uma peça já vinculada a um Job (Projeto). Só os campos informados são alterados.

#### Parâmetros de rota

| Parâmetro   | Tipo    | Obrigatório | Descrição                                                |
|-------------|---------|-------------|-----------------------------------------------------------|
| `id`        | integer | **Sim**     | Identificador do job                                       |
| `jobPecaId` | integer | **Sim**     | ID da peça vinculada ao job (retornado pelo `POST`, não é o id do catálogo) |

#### Exemplo de requisição

```bash
curl -X PUT "https://public-api.iclips.com.br/api/v1/jobs/1234/pecas/42" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Banner Instagram — Revisado",
    "prioridade": 3
  }'
```

#### Campos do Request Body

| Campo        | Tipo           | Obrigatório | Descrição                                |
|--------------|----------------|-------------|--------------------------------------------|
| `titulo`     | string / null  | Não         | Título da peça no job.                     |
| `descricao`  | string / null  | Não         | Descrição da peça no job.                  |
| `formato`    | string / null  | Não         | Formato da peça.                           |
| `servico`    | string / null  | Não         | Nome do serviço.                           |
| `prioridade` | integer / null | Não         | 1=Baixa, 2=Média, 3=Alta.                  |

⚠️ Nenhum campo é obrigatório aqui — envie só os que quer alterar; os demais permanecem
como estavam.

#### Resposta — 200 OK

```json
{
  "jobId": 1234,
  "jobPecaId": 42,
  "pecaId": 5,
  "titulo": "Banner Instagram — Revisado",
  "status": 1
}
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 401  | API Key ausente ou inválida |
| 404  | Job não encontrado, ou `jobPecaId` não pertence a este job |

---

### 11. POST /api/v1/jobs/{id}/tarefas

Cria uma tarefa manual vinculada a um Job (Projeto) já existente.

#### Parâmetros de rota

| Parâmetro | Tipo    | Obrigatório | Descrição            |
|-----------|---------|-------------|----------------------|
| `id`      | integer | **Sim**     | Identificador do job |

#### Exemplo de requisição

```bash
curl -X POST "https://public-api.iclips.com.br/api/v1/jobs/1234/tarefas" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Reunião de briefing",
    "descricao": "Alinhar detalhes com o cliente",
    "inicioDate": "2025-06-02",
    "fimDate": "2025-06-02",
    "prioridade": 1,
    "ordemExibicao": 1
  }'
```

#### Campos do Request Body

Mesmos campos de `tarefasInput[]` do `POST /api/v1/jobs` (seção 6):

| Campo           | Tipo           | Obrigatório | Descrição                                    |
|-----------------|----------------|-------------|-----------------------------------------------|
| `titulo`        | string         | **Sim**     | Título da tarefa. Máx 150 caracteres.          |
| `descricao`     | string / null  | Não         | Descrição da tarefa.                           |
| `inicioDate`    | date / null    | Não         | Data de início da tarefa (`YYYY-MM-DD`).       |
| `fimDate`       | date / null    | Não         | Data de fim da tarefa (`YYYY-MM-DD`).          |
| `prioridade`    | integer / null | Não         | 1=Baixa, 2=Média, 3=Alta.                      |
| `ordemExibicao` | integer / null | Não         | Ordem de exibição da tarefa na lista.          |

#### Resposta — 201 Created

```json
{
  "jobId": 1234,
  "idTarefaJob": 701,
  "titulo": "Reunião de briefing",
  "status": 0
}
```

| Campo         | Tipo    | Descrição             |
|---------------|---------|------------------------|
| `jobId`       | integer | ID do job              |
| `idTarefaJob` | integer | ID da tarefa criada — use este id no `PUT` |
| `titulo`      | string  | Título da tarefa       |
| `status`      | integer | Código de status da tarefa |

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `titulo` vazio ou ausente |
| 401  | API Key ausente ou inválida |
| 404  | Job não encontrado |

---

### 12. PUT /api/v1/jobs/{id}/tarefas/{tarefaId}

Atualiza uma tarefa já vinculada a um Job (Projeto). Só os campos informados são alterados.

#### Parâmetros de rota

| Parâmetro  | Tipo    | Obrigatório | Descrição                                          |
|------------|---------|-------------|------------------------------------------------------|
| `id`       | integer | **Sim**     | Identificador do job                                  |
| `tarefaId` | integer | **Sim**     | ID da tarefa vinculada ao job (retornado pelo `POST`) |

#### Exemplo de requisição

```bash
curl -X PUT "https://public-api.iclips.com.br/api/v1/jobs/1234/tarefas/701" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "titulo": "Reunião de briefing — Remarcada",
    "inicioDate": "2025-06-05"
  }'
```

#### Campos do Request Body

| Campo           | Tipo           | Obrigatório | Descrição                                |
|-----------------|----------------|-------------|---------------------------------------------|
| `titulo`        | string / null  | Não         | Título da tarefa (se enviado, não pode ser vazio). |
| `descricao`     | string / null  | Não         | Descrição da tarefa.                        |
| `inicioDate`    | date / null    | Não         | Data de início da tarefa (`YYYY-MM-DD`).    |
| `fimDate`       | date / null    | Não         | Data de fim da tarefa (`YYYY-MM-DD`).       |
| `prioridade`    | integer / null | Não         | 1=Baixa, 2=Média, 3=Alta.                   |
| `ordemExibicao` | integer / null | Não         | Ordem de exibição da tarefa na lista.       |

⚠️ Nenhum campo é obrigatório aqui — envie só os que quer alterar; os demais permanecem
como estavam.

#### Resposta — 200 OK

```json
{
  "jobId": 1234,
  "idTarefaJob": 701,
  "titulo": "Reunião de briefing — Remarcada",
  "status": 0
}
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `titulo` enviado vazio |
| 401  | API Key ausente ou inválida |
| 404  | Job não encontrado, ou `tarefaId` não pertence a este job |

---

### 13. GET /api/v1/pieces

Lista as peças do catálogo da agência com filtros opcionais e paginação.

#### Parâmetros de filtro (query string)

| Parâmetro           | Tipo    | Obrigatório | Descrição                                |
|---------------------|---------|-------------|------------------------------------------|
| `q`                 | string  | Não         | Busca textual por nome da peça.          |
| `categoria`         | string  | Não         | Filtro por ID de categoria.              |
| `status`            | string  | Não         | Filtro por status da peça.               |
| `basedOnTemplateId` | string  | Não         | Filtro por ID do template base.          |
| `page`              | integer | Não         | Número da página. Padrão: `1`.           |
| `pageSize`          | integer | Não         | Itens por página. Padrão: `20`.          |

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/pieces?q=Banner&page=1&pageSize=20" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

Lista paginada de peças com `id`, `name`, `description`, `exibirBool`, `digitalBool`, `categoriaIds` e metadados de paginação.

---

### 14. GET /api/v1/pieces/{id}

Retorna o detalhe de uma peça, incluindo `workflowJson` e `checklistJson` completos.

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/pieces/456" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 401  | API Key ausente ou inválida |
| 404  | Peça não encontrada |

---

### 15. POST /api/v1/pieces

Cria uma nova peça no catálogo do iClips.

> Use `GET /api/v1/workflow-templates` para descobrir o `id` a usar em `basedOnTemplateId`.
> Use `GET /api/v1/piece-categories` para descobrir os `id`s a usar em `categoriaIds`.

#### Exemplo de requisição

```bash
curl -X POST "https://public-api.iclips.com.br/api/v1/pieces" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Banner Digital Full HD",
    "description": "Banner para veiculação digital em formato Full HD",
    "exibirBool": true,
    "digitalBool": true,
    "isFlexible": false,
    "briefingCliente": "Informe as dimensões e o tema",
    "pecaCliente": true,
    "checklistJson": "{}",
    "categoriaIds": [],
    "basedOnTemplateId": 12
  }'
```

#### Campos do Request Body

| Campo                              | Tipo           | Obrigatório | Descrição                                                                       |
|------------------------------------|----------------|-------------|---------------------------------------------------------------------------------|
| `name`                             | string         | **Sim**     | Nome da peça. Máx 255 caracteres.                                               |
| `description`                      | string / null  | Não         | Descrição da peça.                                                              |
| `valorDecimal`                     | decimal / null | Não         | Valor base.                                                                     |
| `valorSindraproCriacaoDecimal`     | decimal / null | Não         | Valor Sindrapro — Criação.                                                      |
| `valorSindraproAdaptacaoDecimal`   | decimal / null | Não         | Valor Sindrapro — Adaptação.                                                    |
| `valorSindraproFinalizacaoDecimal` | decimal / null | Não         | Valor Sindrapro — Finalização.                                                  |
| `valorTotalDecimal`                | decimal / null | Não         | Valor total.                                                                    |
| `exibirBool`                       | boolean        | Não         | Visível no catálogo. Padrão: `true`.                                            |
| `digitalBool`                      | boolean / null | Não         | Peça digital. Padrão: `true`.                                                   |
| `isFlexible`                       | boolean        | Não         | Peça flexível. Padrão: `false`.                                                 |
| `briefingCliente`                  | string         | Não         | Instrução de briefing ao cliente. Padrão: `""`.                                 |
| `pecaCliente`                      | boolean        | Não         | Padrão: `true`.                                                                 |
| `idSindicato`                      | integer / null | Não         | ID do sindicato vinculado.                                                      |
| `tempoMedio`                       | string / null  | Não         | Tempo médio de produção (ex: `"2h"`).                                           |
| `instrucao`                        | string / null  | Não         | Instrução de produção.                                                          |
| `checklistJson`                    | string         | Não         | JSON do checklist. Padrão: `"{}"`.                                              |
| `categoriaIds`                     | integer[]      | Não         | IDs das categorias. Ver `GET /api/v1/piece-categories`.                         |
| `basedOnTemplateId`                | integer / null | Cond.†      | ID do workflow template base. Ver `GET /api/v1/workflow-templates`.             |
| `workflowJson`                     | string / null  | Cond.†      | JSON do workflow. Mutuamente exclusivo com `basedOnTemplateId`.                 |

†Exatamente um entre `basedOnTemplateId` e `workflowJson` deve ser informado.

#### Resposta — 201 Created

```json
{ "success": true, "data": 456 }
```

`data` = ID da peça criada.

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `name` vazio, ambos ou nenhum de `basedOnTemplateId` / `workflowJson` |
| 401  | API Key ausente ou inválida |

---

### 16. PUT /api/v1/pieces/{id}

Atualiza uma peça existente no catálogo.

#### Exemplo de requisição

```bash
curl -X PUT "https://public-api.iclips.com.br/api/v1/pieces/456" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Banner Digital Full HD — v2",
    "exibirBool": true,
    "pecaCliente": true,
    "workflowJson": "{}",
    "checklistJson": "{}",
    "categoriaIds": []
  }'
```

#### Campos do Request Body

Mesmos campos do POST, com as seguintes obrigatoriedades adicionais:

| Campo           | Tipo   | Obrigatório | Descrição          |
|-----------------|--------|-------------|--------------------|
| `name`          | string | **Sim**     | Nome da peça.      |
| `workflowJson`  | string | **Sim**     | JSON do workflow.  |
| `checklistJson` | string | **Sim**     | JSON do checklist. |

#### Resposta — 200 OK

```json
{ "success": true, "data": true }
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `name`, `workflowJson` ou `checklistJson` ausentes |
| 401  | API Key ausente ou inválida |
| 404  | Peça não encontrada |

---

### 17. PATCH /api/v1/pieces/{id}/checklist

Marca ou desmarca um item do checklist de uma peça.

#### Exemplo de requisição

```bash
curl -X PATCH "https://public-api.iclips.com.br/api/v1/pieces/456/checklist" \
  -H "X-Api-Key: <SUA_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "itemId": "step-1",
    "done": true
  }'
```

#### Campos do Request Body

| Campo    | Tipo    | Obrigatório | Descrição                               |
|----------|---------|-------------|-----------------------------------------|
| `itemId` | string  | **Sim**     | Identificador do item no checklist.     |
| `done`   | boolean | **Sim**     | `true` = concluído, `false` = pendente. |

#### Resposta — 200 OK

```json
{ "success": true, "data": true }
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 400  | `itemId` vazio |
| 401  | API Key ausente ou inválida |
| 404  | Peça não encontrada |

---

### 18. DELETE /api/v1/pieces/{id}

Exclui (soft-delete) uma peça do catálogo.

#### Exemplo de requisição

```bash
curl -X DELETE "https://public-api.iclips.com.br/api/v1/pieces/456" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{ "success": true, "data": true }
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 401  | API Key ausente ou inválida |
| 404  | Peça não encontrada |

---

### 19. GET /api/v1/workflow-templates

Lista os workflow templates disponíveis na agência (sem o `workflowJson` completo).
Use o `id` retornado no campo `basedOnTemplateId` ao criar peças via `POST /api/v1/pieces`.

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/workflow-templates" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

Array de templates com `id`, `nome` e `descricao` (sem `workflowJson`).

---

### 20. GET /api/v1/workflow-templates/{id}

Retorna o detalhe de um workflow template, incluindo o `workflowJson` completo.

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/workflow-templates/12" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Erros

| HTTP | Cenário |
|------|---------|
| 401  | API Key ausente ou inválida |
| 404  | Template não encontrado |

---

### 21. GET /api/v1/piece-categories

Lista as categorias de peça disponíveis na agência.
Use os `id`s retornados no campo `categoriaIds` ao criar ou atualizar peças.

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/piece-categories" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

Array de categorias com `id` e `nome`.

---

### 22. GET /api/v1/piece-categories/dropdown

Retorna categorias de peça em formato simplificado para uso em dropdowns.

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/piece-categories/dropdown" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

Array simplificado de categorias com `id` e `nome`.

---

### 23. GET /api/v1/projetos

Retorna a hierarquia completa de Projetos (Jobs → Peças → Workflows → Atividades e Tarefas → Atividades) para o período informado. Ideal para integrações de BI, exportações e sincronização com ferramentas externas.

> **Limite de taxa:** 10 requisições por minuto por API Key. Excedido, a API retorna **HTTP 429** com o header `Retry-After` indicando quantos segundos aguardar.

#### Parâmetros de filtro (query string)

| Parâmetro        | Tipo     | Obrigatório | Descrição                                                                             |
|------------------|----------|-------------|---------------------------------------------------------------------------------------|
| `dataInicio`     | datetime | **Sim**     | Data/hora de início do período (`YYYY-MM-DDTHH:mm:ss`). Filtra pela data de entrada do job. |
| `dataFim`        | datetime | **Sim**     | Data/hora de fim do período. Máximo de 31 dias de diferença em relação a `dataInicio`. |
| `idCliente`      | integer  | Não         | Filtra pelo ID do cliente vinculado ao job.                                           |
| `idGrupoCliente` | integer  | Não         | Filtra pelo ID do grupo de clientes vinculado ao job.                                 |
| `page`           | integer  | Não         | Número da página. Padrão: `1`. Mínimo: `1`.                                          |
| `pageSize`       | integer  | Não         | Itens por página. Padrão: `20`. Máximo: `50`.                                        |

#### Exemplo de requisição

```bash
curl -X GET "https://public-api.iclips.com.br/api/v1/projetos?dataInicio=2025-01-01T00:00:00&dataFim=2025-01-31T23:59:59&page=1&pageSize=20" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

#### Resposta — 200 OK

```json
{
  "data": [
    {
      "idProjeto": 1234,
      "nomeProjeto": "Campanha Verão 2025",
      "statusProjeto": 6,
      "verba": 50000.00,
      "datas": {
        "entrada": "2025-01-05T00:00:00",
        "aprovacao": "2025-01-10T00:00:00",
        "conclusaoEstimada": "2025-01-31T00:00:00",
        "conclusao": null,
        "alteracaoStatus": "2025-01-10T09:30:00"
      },
      "responsaveis": {
        "principal": {
          "id": 42,
          "nome": "Ana Silva",
          "cpf": "123.456.789-00"
        },
        "auxiliar": null
      },
      "cliente": {
        "id": 10,
        "nome": "Empresa Contratante S/A",
        "cnpj": "11.222.333/0001-44"
      },
      "grupoCliente": null,
      "pecas": [
        {
          "idJobPeca": 501,
          "idPeca": 88,
          "nomePeca": "Banner Full HD",
          "tituloAtividade": "Produção Banner",
          "status": 2,
          "isFlexible": false,
          "inicioPlanejado": "2025-01-12T00:00:00",
          "fimPlanejado": "2025-01-20T00:00:00",
          "workflows": [
            {
              "idWorkflow": 301,
              "nome": "Criação",
              "refacao": false,
              "tempoEstimado": "04:00",
              "inicio": "2025-01-12T08:00:00",
              "fim": "2025-01-15T18:00:00",
              "atividades": [
                {
                  "inicioPlay": "2025-01-13T09:00:00",
                  "fimPlay": "2025-01-13T13:00:00",
                  "statusConclusao": null,
                  "tempoGasto": "04:00",
                  "executor": {
                    "id": 55,
                    "nome": "Carlos Lima",
                    "cpf": "987.654.321-00",
                    "departamento": "Criação",
                    "valorHora": 85.00
                  }
                }
              ]
            }
          ]
        }
      ],
      "tarefas": [
        {
          "idTarefaJob": 701,
          "tituloAtividade": "Aprovação cliente",
          "tempoEstimado": "01:00",
          "inicioPlanejado": "2025-01-25T00:00:00",
          "fimPlanejado": "2025-01-26T00:00:00",
          "atividades": []
        }
      ]
    }
  ],
  "meta": {
    "totalCount": 48,
    "page": 1,
    "pageSize": 20
  }
}
```

#### Campos da resposta

**Projeto (`data[]`)**

| Campo                          | Tipo            | Descrição                                                      |
|--------------------------------|-----------------|----------------------------------------------------------------|
| `idProjeto`                    | integer         | Identificador único do job/projeto                             |
| `nomeProjeto`                  | string / null   | Título do projeto                                              |
| `statusProjeto`                | integer / null  | Código do status (mesmos valores do `POST /api/v1/jobs`)       |
| `verba`                        | decimal / null  | Verba disponível do projeto                                    |
| `datas.entrada`                | datetime / null | Data de entrada do projeto                                     |
| `datas.aprovacao`              | datetime / null | Data de aprovação                                              |
| `datas.conclusaoEstimada`      | datetime / null | Data prevista de conclusão                                     |
| `datas.conclusao`              | datetime / null | Data efetiva de conclusão (somente se `statusProjeto` = `7`)   |
| `datas.alteracaoStatus`        | datetime / null | Data da última alteração de status                             |
| `responsaveis.principal`       | object / null   | Responsável principal pelo projeto                             |
| `responsaveis.auxiliar`        | object / null   | Responsável auxiliar pelo projeto                              |
| `responsaveis.*.id`            | integer / null  | ID do funcionário                                              |
| `responsaveis.*.nome`          | string / null   | Nome do funcionário                                            |
| `responsaveis.*.cpf`           | string / null   | CPF do funcionário                                             |
| `cliente`                      | object / null   | Cliente vinculado ao projeto                                   |
| `grupoCliente`                 | object / null   | Grupo de clientes vinculado (mutuamente exclusivo com cliente)  |
| `cliente.id` / `grupoCliente.id` | integer / null | ID do cliente ou grupo                                        |
| `cliente.nome` / `grupoCliente.nome` | string / null | Nome do cliente ou grupo                                  |
| `cliente.cnpj`                 | string / null   | CNPJ do cliente (apenas em `cliente`)                          |

**Peça (`pecas[]`)**

| Campo              | Tipo            | Descrição                                                  |
|--------------------|-----------------|------------------------------------------------------------|
| `idJobPeca`        | integer         | ID da instância da peça no job                             |
| `idPeca`           | integer / null  | ID do catálogo de peças                                    |
| `nomePeca`         | string / null   | Nome da peça                                               |
| `tituloAtividade`  | string / null   | Título da atividade da peça                                |
| `status`           | integer / null  | Status da peça no job                                      |
| `isFlexible`       | boolean         | Indica se a peça é flexível                                |
| `inicioPlanejado`  | datetime / null | Data de início planejada                                   |
| `fimPlanejado`     | datetime / null | Data de fim planejada                                      |

**Workflow (`pecas[].workflows[]`)**

| Campo           | Tipo            | Descrição                                              |
|-----------------|-----------------|--------------------------------------------------------|
| `idWorkflow`    | integer         | ID do workflow na peça                                 |
| `nome`          | string / null   | Nome da etapa de workflow                              |
| `refacao`       | boolean         | Indica se é uma refação                                |
| `tempoEstimado` | string / null   | Tempo estimado no formato `HH:mm`                      |
| `inicio`        | datetime / null | Data de início do workflow                             |
| `fim`           | datetime / null | Data de fim do workflow                                |

**Atividade (`workflows[].atividades[]` e `tarefas[].atividades[]`)**

| Campo            | Tipo            | Descrição                                              |
|------------------|-----------------|--------------------------------------------------------|
| `inicioPlay`     | datetime / null | Início do apontamento de hora                          |
| `fimPlay`        | datetime / null | Fim do apontamento de hora                             |
| `statusConclusao`| datetime / null | Data de conclusão, quando concluída                    |
| `tempoGasto`     | string / null   | Tempo gasto no formato `HH:mm`                         |
| `executor.id`    | integer / null  | ID do funcionário que executou                         |
| `executor.nome`  | string / null   | Nome do executor                                       |
| `executor.cpf`   | string / null   | CPF do executor                                        |
| `executor.departamento` | string / null | Departamento do executor                          |
| `executor.valorHora`    | decimal / null | Valor/hora do executor                           |

**Tarefa (`tarefas[]`)**

| Campo              | Tipo            | Descrição                                              |
|--------------------|-----------------|--------------------------------------------------------|
| `idTarefaJob`      | integer         | ID da tarefa no job                                    |
| `tituloAtividade`  | string / null   | Título da tarefa                                       |
| `tempoEstimado`    | string / null   | Tempo estimado no formato `HH:mm`                      |
| `inicioPlanejado`  | datetime / null | Data de início planejada                               |
| `fimPlanejado`     | datetime / null | Data de fim planejada                                  |
| `atividades`       | array           | Apontamentos de hora realizados nesta tarefa           |

**Meta (`meta`)**

| Campo        | Tipo    | Descrição                                      |
|--------------|---------|------------------------------------------------|
| `totalCount` | integer | Total de projetos no período filtrado          |
| `page`       | integer | Página atual                                   |
| `pageSize`   | integer | Itens por página efetivamente aplicados        |

#### Resposta — 400 Bad Request

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "O intervalo de datas não pode exceder 31 dias.",
    "details": [
      { "field": "date_range", "issue": "exceeded" }
    ]
  }
}
```

| HTTP | Cenário                                                                        |
|------|--------------------------------------------------------------------------------|
| 400  | `dataInicio` ou `dataFim` ausentes, `dataInicio` > `dataFim`, intervalo > 31 dias |
| 401  | API Key ausente ou inválida                                                    |
| 429  | Limite de taxa excedido (10 req/min). Header `Retry-After` indica segundos restantes. |

---

## Paginação

Os endpoints de listagem suportam paginação via parâmetros `limit` e `offset` (financeiro) ou `page` e `pageSize` (projetos/peças).

**Exemplo — navegar por todas as páginas (financeiro):**

```bash
# Página 1
curl "https://public-api.iclips.com.br/api/v1/financial/entries?from=2025-01-01&to=2025-03-31&limit=50&offset=0" \
  -H "X-Api-Key: <SUA_API_KEY>"

# Página 2
curl "https://public-api.iclips.com.br/api/v1/financial/entries?from=2025-01-01&to=2025-03-31&limit=50&offset=50" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

Continue iterando enquanto `meta.hasMore` for `true` ou até que `meta.offset + data.length >= meta.total`.

---

## Limites e boas práticas

| Restrição                                                                              | Valor    |
|----------------------------------------------------------------------------------------|----------|
| Período máximo por consulta (financeiro)                                               | 366 dias |
| Itens por página — financeiro (máximo)                                                 | 200      |
| Itens por página — financeiro (padrão)                                                 | 50       |
| Itens por página — peças (padrão)                                                      | 20       |
| Os parâmetros `from` e `to` são **obrigatórios** em todos os endpoints financeiros de listagem (exceto `/accounts`) | —        |
| Período máximo por consulta (projetos)                                                                               | 31 dias  |
| Itens por página — projetos (máximo)                                                                                 | 50       |
| Itens por página — projetos (padrão)                                                                                 | 20       |
| Limite de taxa — projetos (`GET /api/v1/projetos`)                                                                   | 10 req/min |

---

*Versão da API: v1 — Última atualização: Junho de 2026*
