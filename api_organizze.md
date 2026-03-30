# API Organizze v2 - Documentação

## Introdução

A API do Organizze possibilita que aplicações se comuniquem com a sua conta no sistema. Esta é a primeira versão da API, ainda em versão beta.

---

## Autenticação

Todas as requisições utilizam **HTTP Basic Auth**:

| Campo    | Valor                                                                 |
|----------|-----------------------------------------------------------------------|
| Username | Email da conta do Organizze                                           |
| Password | Token de acesso (disponível em `https://app.organizze.com.br/configuracoes/api-keys`) |

- Apenas **HTTPS** é aceito
- URL base: `https://api.organizze.com.br/rest/v2`
- O header `User-Agent` é **obrigatório** em todas as requisições

**Exemplos de User-Agent:**
```
User-Agent: Esdras (esdras@organizze.com.br)
User-Agent: Alex (alex@gmail.com)
```

---

## Formato

A API suporta apenas **JSON**. Todas as respostas são em `application/json; charset=utf-8`.

---

## Paginação

Movimentações e faturas de cartão de crédito são paginadas por período usando os parâmetros:

```
?start_date=2015-09-01&end_date=2015-09-30
```

- Sem parâmetros: retorna o **mês atual** para movimentações e o **ano atual** para faturas
- O período processado é sempre mês inteiro (`beginning_of_month` / `end_of_month`)

---

## Erros

### 401 - Não autorizado
```json
{
  "error": "Não autorizado"
}
```

### 422 - Registro inválido
```json
{
  "id": null,
  "name": null,
  "errors": {
    "name": ["não pode estar em branco"]
  }
}
```

---

## Endpoints

### Usuários

#### Detalhar usuário
```
GET /users/{id}
```

**Response:**
```json
{
  "id": 3,
  "name": "Esdras Mayrink",
  "email": "falecom@email.com.br",
  "role": "admin"
}
```

---

### Contas Bancárias

#### Listar contas
```
GET /accounts
```

#### Detalhar conta
```
GET /accounts/{id}
```

#### Criar conta
```
POST /accounts
```
```json
{
  "name": "Itaú CC",
  "type": "checking",
  "description": "Minha conta corrente",
  "default": true
}
```

Tipos disponíveis para `type`: `checking`, `savings`, `other`

#### Atualizar conta
```
PUT /accounts/{id}
```

#### Excluir conta
```
DELETE /accounts/{id}
```

---

### Categorias

#### Listar categorias
```
GET /categories
```

#### Detalhar categoria
```
GET /categories/{id}
```

#### Criar categoria
```
POST /categories
```
```json
{
  "name": "SEO"
}
```

#### Atualizar categoria
```
PUT /categories/{id}
```

#### Excluir categoria
```
DELETE /categories/{id}
```

Ao excluir, é possível informar uma categoria substituta. Todas as movimentações da categoria excluída serão migradas para ela. Se não informada, a categoria padrão é usada.

```json
{
  "replacement_id": 18
}
```

---

### Metas (Budgets)

#### Listar metas

| Escopo          | Endpoint              |
|-----------------|-----------------------|
| Mês atual       | `GET /budgets`        |
| Ano inteiro     | `GET /budgets/2018`   |
| Mês específico  | `GET /budgets/2018/08`|

**Response:**
```json
[
  {
    "amount_in_cents": 150000,
    "category_id": 17,
    "date": "2018-08-01",
    "activity_type": 0,
    "total": 0,
    "predicted_total": 0,
    "percentage": "0.0"
  }
]
```

---

### Cartões de Crédito

#### Listar cartões
```
GET /credit_cards
```

#### Detalhar cartão
```
GET /credit_cards/{id}
```

#### Criar cartão
```
POST /credit_cards
```
```json
{
  "name": "Hipercard",
  "card_network": "hipercard",
  "due_day": 15,
  "closing_day": 2,
  "limit_cents": 500000
}
```

#### Atualizar cartão
```
PUT /credit_cards/{id}
```

O campo `update_invoices_since` pode ser enviado para recalcular faturas a partir de uma data:
```json
{
  "update_invoices_since": "2015-07-01"
}
```

#### Excluir cartão
```
DELETE /credit_cards/{id}
```

---

### Faturas de Cartão de Crédito

#### Listar faturas de um cartão
```
GET /credit_cards/{id}/invoices
```

Paginadas por período (`start_date` / `end_date`). Sem parâmetros, retorna o ano atual.

#### Detalhar fatura
```
GET /credit_cards/{id}/invoices/{invoice_id}
```

Retorna a fatura com seus `transactions` e `payments`.

#### Pagamento de uma fatura
```
GET /credit_cards/{id}/invoices/{invoice_id}/payments
```

---

### Movimentações (Transactions)

#### Listar movimentações
```
GET /transactions?start_date=2015-09-01&end_date=2015-09-30
```

Filtro opcional por conta bancária:
```
GET /transactions?account_id={id}
```

#### Detalhar movimentação
```
GET /transactions/{id}
```

#### Criar movimentação simples
```
POST /transactions
```
```json
{
  "description": "Computador",
  "notes": "Pagamento via boleto",
  "date": "2015-09-16",
  "amount_cents": -150000,
  "account_id": 3,
  "category_id": 21,
  "tags": [{"name": "homeoffice"}]
}
```

#### Criar movimentação recorrente (fixa)
```
POST /transactions
```
```json
{
  "description": "Despesa fixa",
  "date": "2015-09-16",
  "recurrence_attributes": {"periodicity": "monthly"}
}
```

Valores válidos para `periodicity`: `monthly`, `yearly`, `weekly`, `biweekly`, `bimonthly`, `trimonthly`

#### Criar movimentação parcelada
```
POST /transactions
```
```json
{
  "description": "Despesa parcelada",
  "date": "2015-09-16",
  "installments_attributes": {"periodicity": "monthly", "total": 12}
}
```

#### Atualizar movimentação
```
PUT /transactions/{id}
```

Para movimentações fixas ou parceladas:

| Atributo        | Efeito                                             |
|-----------------|----------------------------------------------------|
| `update_future` | Atualiza esta e as próximas ocorrências            |
| `update_all`    | Atualiza todas as ocorrências (inclusive passadas) |

> Atenção: `update_all` pode alterar o saldo da conta se ocorrências anteriores já estiverem pagas.

#### Excluir movimentação
```
DELETE /transactions/{id}
```

Os mesmos atributos `update_future` e `update_all` se aplicam ao body da exclusão.

---

### Transferências (Transfers)

Registra uma transferência entre duas contas bancárias. Cria dois registros: saída na conta de origem e entrada na conta de destino.

> Cartões de crédito **não** são aceitos como conta de origem ou destino.

#### Listar transferências
```
GET /transfers
```

#### Detalhar transferência
```
GET /transfers/{id}
```

#### Criar transferência
```
POST /transfers
```
```json
{
  "credit_account_id": 3,
  "debit_account_id": 4,
  "amount_cents": 10000,
  "date": "2015-09-01",
  "paid": true,
  "tags": [{"name": "ajuste"}]
}
```

#### Atualizar transferência
```
PUT /transfers/{id}
```

#### Excluir transferência
```
DELETE /transfers/{id}
```

---

## Campos Comuns em Movimentações

| Campo                       | Descrição                                                       |
|-----------------------------|-----------------------------------------------------------------|
| `amount_cents`              | Valor em centavos (negativo = despesa, positivo = receita)      |
| `paid`                      | Se a movimentação está paga/recebida                            |
| `recurring`                 | Se é uma movimentação recorrente                                |
| `total_installments`        | Total de parcelas                                               |
| `installment`               | Número da parcela atual                                         |
| `account_id`                | ID da conta bancária                                            |
| `credit_card_id`            | ID do cartão (quando lançado no cartão)                         |
| `credit_card_invoice_id`    | ID da fatura vinculada                                          |
| `oposite_transaction_id`    | ID da transação oposta (em caso de transferência)               |
| `oposite_account_id`        | ID da conta oposta (em caso de transferência)                   |
| `tags`                      | Array de tags: `[{"name": "tag"}]`                              |