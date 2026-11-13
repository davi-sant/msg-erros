
## Descrição

Ao realizar o orçamento de um item cuja unidade é **KG** (ou **LT** / **TO**), com o parâmetro **"digitar valor total"** habilitado, o sistema apresenta comportamento incorreto no cálculo entre **quantidade** e **valor total**.

## Comportamento Esperado

Quando o parâmetro de digitar valor total está habilitado:

1. O foco deve ir direto para o campo **Valor Total** (comportamento atual — correto).
2. Ao informar o **Valor Total**, o sistema deve **calcular a quantidade** com base no preço unitário do produto:
   - `quantidade = valor_total / preço_unitário`
3. O inverso também deve funcionar: ao informar uma **quantidade**, o sistema deve recalcular o **valor total** com base no preço.

## Comportamento Atual (Bug)

### Cenário 1 — Informar Valor Total
- Ao digitar o **valor total**, o sistema **não calcula a quantidade**.
- A quantidade permanece fixa em **1**.
- O valor total é tratado como se fosse o **preço unitário**.
- **Resultado:** quantidade e valor total inconsistentes com o preço do produto.

### Cenário 2 — Informar Quantidade Fracionada
- Ao informar uma **quantidade fracionada** (ex.: `1,5`) em um produto cuja embalagem é **UN** (que não deveria aceitar fração), o sistema:
  - Aceita a fração indevidamente, **ou**
  - Exibe mensagem de **"desconto aplicado"**, mesmo quando **não há desconto** sendo concedido.
- **Resultado:** mensagem enganosa e validação de unidade inadequada.

## Passos para Reproduzir

### Caso 1
1. Criar/editar um orçamento com item de unidade **KG**.
2. Garantir que o parâmetro **"digitar valor total em produtos KG, LT, TO"** esteja habilitado.
3. No item, informar apenas o **valor total** (ex.: `R$ 50,00`) para um produto com preço unitário conhecido (ex.: `R$ 10,00/kg`).
4. **Observar:** quantidade permanece `1` e valor total `50,00`, quando o esperado seria quantidade `5`.

### Caso 2
1. Criar/editar um orçamento com item de embalagem **UN**.
2. Informar uma **quantidade fracionada** (ex.: `1,5`).
3. **Observar:** sistema aceita a fração e/ou exibe mensagem de desconto inexistente.

## Impacto

- **Cálculos incorretos** de quantidade e valor em orçamentos com itens KG/LT/TO.
- **Mensagens enganosas** ao usuário (desconto inexistente).
- **Validação de unidade falha** (permite fração em UN).
- Risco de **orçamentos com valores errados** sendo enviados a clientes.

## Correção Sugerida

1. **Ao digitar valor total (KG/LT/TO):** calcular `quantidade = valor_total / preço_unitário` e atualizar o campo quantidade.
2. **Ao digitar quantidade:** recalcular `valor_total = quantidade × preço_unitário`.
3. **Validação de unidade:** bloquear quantidades fracionadas para produtos com embalagem **UN** (ou tratar conforme regra de negócio).
4. **Mensagem de desconto:** só exibir quando houver **diferença real** entre o valor calculado e o valor informado (desconto efetivo), nunca em recálculo normal de quantidade/preço.

## Ambiente

- Módulo: Orçamento
- Parâmetro envolvido: *"Digitar valor total em produtos KG, LT, TO"*
- Unidades afetadas: **KG**, **LT**, **TO** (cálculo) e **UN** (validação de fração)

## Prioridade

**Alta** — afeta cálculo financeiro direto do orçamento e pode gerar propostas com valores incorretos.

---
