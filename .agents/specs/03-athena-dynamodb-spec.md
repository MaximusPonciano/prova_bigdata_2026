# Spec 03: Athena + DynamoDB + Entrega

## Objetivo
Validar o pipeline de dados executando as consultas Athena de referência, conferindo os metadados no DynamoDB e preparando a entrega final (fork, branch, PR).

## Parte A — Consultas Athena de Referência

### Pré-requisito: Glue Data Catalog
As tabelas do gold devem estar registradas no **Glue Data Catalog** (via Terraform ou Crawler) para que o Athena possa lê-las:
- `fato_pedidos` — com coluna de partição `data_pedido`.
- `dim_cliente` — sem partição.
- `dim_produto` — sem partição.

### Pré-requisito: Athena Workgroup
O Athena Workgroup deve ter um `output_location` configurado no S3 (ex: `s3://<gold>/athena-results/`).

### 4 Consultas de Referência

#### Consulta 1 — Faturamento por categoria
```sql
SELECT p.categoria, ROUND(SUM(f.valor_total), 2) AS faturamento
FROM fato_pedidos f
JOIN dim_produto p ON f.produto_id = p.produto_id
GROUP BY p.categoria
ORDER BY faturamento DESC;
```
**Critério:** uma linha por categoria existente; sem nulos em `categoria` (nulos viraram `DESCONHECIDO`). Soma total = `SUM(valor_total)` de `fato_pedidos`.

#### Consulta 2 — Top 5 clientes por gasto
```sql
SELECT c.cliente_nome, ROUND(SUM(f.valor_total), 2) AS gasto_total
FROM fato_pedidos f
JOIN dim_cliente c ON f.cliente_id = c.cliente_id
GROUP BY c.cliente_nome
ORDER BY gasto_total DESC
LIMIT 5;
```
**Critério:** no máximo 5 linhas, ordenadas decrescentemente por `gasto_total`.

#### Consulta 3 — Pedidos e faturamento por dia (partição)
```sql
SELECT data_pedido, COUNT(*) AS qtd_pedidos, ROUND(SUM(valor_total), 2) AS faturamento_dia
FROM fato_pedidos
GROUP BY data_pedido
ORDER BY data_pedido;
```
**Critério:** uma linha por `data_pedido` distinta; `SUM(qtd_pedidos)` = total de linhas de `fato_pedidos`.

#### Consulta 4 — Verificação de integridade (bônus)
```sql
SELECT COUNT(*) AS orfaos
FROM fato_pedidos f
LEFT JOIN dim_produto p ON f.produto_id = p.produto_id
WHERE p.produto_id IS NULL;
```
**Critério:** `orfaos = 0` (integridade referencial fato → dimensão).

### Evidências
Salvar os resultados de cada consulta em `entregas/<RA>/athena-queries/`.

---

## Parte B — Catálogo de Metadados no DynamoDB

### Schema do Item de Metadados
| Atributo | Tipo | Significado |
|----------|------|-------------|
| `execution_id` | S (PK) | ID único da execução (UUID ou timestamp + random) |
| `data_hora` | S | Data/hora ISO-8601 da execução |
| `dataset` | S | Nome do dataset processado (ex.: `pedidos_desnormalizado`) |
| `linhas_lidas` | N | Linhas lidas do Bucket_Raw |
| `linhas_gravadas` | N | Linhas gravadas no Bucket_Gold (fato) |
| `status` | S | `SUCESSO` \| `FALHA` |

### Validação
- Após execução bem-sucedida do Glue Job, o DynamoDB deve conter **pelo menos 1 item** com `status = "SUCESSO"`.
- Verificar via console ou CLI: `aws dynamodb scan --table-name execucoes --region us-east-1`.
- Salvar evidência em `entregas/<RA>/dynamodb-item.txt`.

---

## Parte C — Fluxo de Entrega

### Passos Obrigatórios
1. **Fork** do repositório do professor na conta do aluno.
2. Criar **branch** no padrão `prova-SEURA` (ex: `prova-2500123`).
3. Colocar os arquivos em `entregas/<RA>/`:
   - Cópia do `infra/gold.tf` (e outros `.tf` criados).
   - Cópia do `glue-job/normaliza_pedidos.py` preenchido.
   - Evidências de execução (terraform, Glue, Athena, DynamoDB, destroy).
4. **Commit + Push** com Conventional Commits.
5. Abrir **Pull Request** para a `master` do repositório original.
6. Antes do PR: percorrer o **`CHECKLIST.md`** item por item.

### Limpeza Final (OBRIGATÓRIA)
```bash
cd infra
terraform destroy
```
Guardar a evidência do destroy em `entregas/<RA>/terraform-destroy.txt`.

---

## Critério de Nota (RUBRICA.md)
- **Critério 4:** Consultas Athena retornam os resultados esperados — **15 pontos**.
- **Critério 5:** Metadados no DynamoDB conforme schema — **10 pontos**.
- **Critério 6:** Custo/limpeza/segurança (tags, destroy, buckets privados, LabRole) — **15 pontos**.
