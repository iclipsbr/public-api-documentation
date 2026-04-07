# iClips Public API — Módulo Financeiro

Documentação de referência para integração com os endpoints financeiros da iClips Public API.

**Base URL:** `https://public-api.iclips.com.br`

---

## Índice

- [Autenticação](#autenticação)
- [Formato das respostas](#formato-das-respostas)
- [Erros](#erros)
- [Endpoints](#endpoints)
  - [GET /api/v1/financial/accounts — Listar contas bancárias](#1-get-apiv1financialaccounts)
  - [GET /api/v1/financial/entries — Listar lançamentos](#2-get-apiv1financialentries)
  - [GET /api/v1/financial/entries/{id} — Detalhe do lançamento](#3-get-apiv1financialentriesid)
  - [GET /api/v1/financial/invoices — Listar notas fiscais](#4-get-apiv1financialinvoices)
  - [GET /api/v1/financial/invoices/{id} — Detalhe da nota fiscal](#5-get-apiv1financialinvoicesid)
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

## Paginação

Os endpoints de listagem suportam paginação via parâmetros `limit` e `offset`.

**Exemplo — navegar por todas as páginas:**

```bash
# Página 1
curl "https://public-api.iclips.com.br/api/v1/financial/entries?from=2025-01-01&to=2025-03-31&limit=50&offset=0" \
  -H "X-Api-Key: <SUA_API_KEY>"

# Página 2
curl "https://public-api.iclips.com.br/api/v1/financial/entries?from=2025-01-01&to=2025-03-31&limit=50&offset=50" \
  -H "X-Api-Key: <SUA_API_KEY>"

# Página 3
curl "https://public-api.iclips.com.br/api/v1/financial/entries?from=2025-01-01&to=2025-03-31&limit=50&offset=100" \
  -H "X-Api-Key: <SUA_API_KEY>"
```

Continue iterando enquanto `meta.hasMore` for `true` ou até que `meta.offset + data.length >= meta.total`.

---

## Limites e boas práticas

| Restrição                                                                              | Valor    |
|----------------------------------------------------------------------------------------|----------|
| Período máximo por consulta                                                            | 366 dias |
| Itens por página (máximo)                                                              | 200      |
| Itens por página (padrão)                                                              | 50       |
| Os parâmetros `from` e `to` são **obrigatórios** em todos os endpoints de listagem (exceto `/accounts`) | —        |

---

*Versão da API: v1 — Última atualização: Abril de 2026*
