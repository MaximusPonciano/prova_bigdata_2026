# Spec 02: Job PySpark de Normalização

## Objetivo
Implementar o Job de normalização em `glue-job/normaliza_pedidos.py`, transformando o CSV desnormalizado (`pedidos_desnormalizado.csv`) em um **esquema estrela** com 1 fato + 2 dimensões, gravando Parquet particionado no Bucket_Gold e registrando metadados no DynamoDB.

## Arquivo Alvo
- `glue-job/normaliza_pedidos.py` — esqueleto com `TODO(aluno)` e `NotImplementedError`.
- O aluno deve preencher as funções: `normalizar()`, `montar_metadados()`, `ler_raw()`, `escrever_gold()`, `gravar_metadados_dynamo()`.

## Contrato de 6 Passos (design do Job)
1. **Ler argumentos do Glue** — `getResolvedOptions`: `RAW_PATH`, `GOLD_PATH`, `DDB_TABLE`, `DATASET_NAME`.
2. **Ler o CSV do RAW_PATH** — `header=True`, `inferSchema=True`. Contar `linhas_lidas`.
3. **Tratar nulos/dados inválidos** (Req 6.7):
   - **Descartar do fato:** linhas sem `pedido_id`, `cliente_id` ou `produto_id` (nulos ou vazios), ou com `quantidade` ausente / `<= 0`.
   - **Nas dimensões:** textos ausentes (`cliente_nome`, `cliente_uf`, `produto_nome`, `categoria`) → `"DESCONHECIDO"` (a linha é mantida).
4. **Normalizar** em `fato_pedidos` + `dim_cliente` + `dim_produto`.
5. **Gravar Parquet** particionado no `GOLD_PATH`:
   - `fato_pedidos` particionado por `data_pedido`: `s3://<gold>/fato_pedidos/data_pedido=YYYY-MM-DD/part-*.parquet`.
   - `dim_cliente` e `dim_produto` **sem partição**: `s3://<gold>/dim_cliente/` e `s3://<gold>/dim_produto/`.
6. **Montar e gravar metadados** no DynamoDB (`execution_id`, `data_hora`, `dataset`, `linhas_lidas`, `linhas_gravadas`, `status`).

## Modelo Dimensional Alvo (Esquema Estrela)

### `dim_cliente` (uma linha por `cliente_id`)
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `cliente_id` | string (PK) | Identificador único do cliente |
| `cliente_nome` | string | Nome do cliente (nulo → `DESCONHECIDO`) |
| `cliente_uf` | string | UF do cliente (nulo → `DESCONHECIDO`) |

### `dim_produto` (uma linha por `produto_id`)
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `produto_id` | string (PK) | Identificador único do produto |
| `produto_nome` | string | Nome do produto (nulo → `DESCONHECIDO`) |
| `categoria` | string | Categoria do produto (nulo → `DESCONHECIDO`) |

### `fato_pedidos` (uma linha por `pedido_id`)
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `pedido_id` | string (PK) | Identificador único do pedido |
| `data_pedido` | date | Data do pedido (coluna de partição) |
| `cliente_id` | string (FK) | FK → `dim_cliente` |
| `produto_id` | string (FK) | FK → `dim_produto` |
| `preco_unitario` | double | Preço unitário do produto |
| `quantidade` | int | Quantidade comprada (> 0) |
| `valor_total` | double | preco_unitario × quantidade |

## Arquitetura do Script (Funções Puras vs I/O)
- **Funções puras** (testáveis localmente sem AWS):
  - `normalizar(df_raw)` → retorna `{"fato_pedidos": ..., "dim_cliente": ..., "dim_produto": ...}`.
  - `montar_metadados(execution_id, dataset, linhas_lidas, linhas_gravadas, status)` → retorna `dict`.
- **Funções de I/O** (efeitos colaterais, só rodam no Glue/AWS):
  - `ler_raw(spark, raw_path)` → lê CSV do S3.
  - `escrever_gold(tabelas, gold_path)` → grava Parquet no S3.
  - `gravar_metadados_dynamo(item, ddb_table)` → grava no DynamoDB via `boto3`.

## Validação Local
- Rodar na pasta `local-test/`:
  ```bash
  docker build -t prova-local . && docker run --rm prova-local
  ```
- Todos os **6 testes de propriedade** (Hypothesis, ≥ 100 iterações) + **4 testes de borda** devem passar.
- Os testes exercitam as funções puras contra a implementação de referência.

## Critério de Nota (RUBRICA.md)
- **Critério 2:** Normalização correta (fato + 2 dimensões, integridade, tratamento de nulos) — **25 pontos**.
- **Critério 3:** Parquet particionado por `data_pedido` — **15 pontos**.
