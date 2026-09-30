---
name: collect-evidence-bigdata
description: >-
  Use esta skill para automatizar a captura de evidências (prints e logs)
  necessárias para a entrega da prova de Big Data.
---

# Coleta de Evidências (Big Data)

Durante as fases de validação, você precisará gerar prints/logs e guardá-los na pasta `entregas/<RA>/`.
Sempre crie o diretório `entregas/<RA>/` se não existir.

## Evidências Necessárias (baseado na RUBRICA.md):

### 1. Infraestrutura (Critério 1 — 20pts)
- **`terraform-apply.txt`**: Saída completa do `terraform apply` provando que a infra foi criada sem erros.
  ```bash
  cd infra
  terraform apply 2>&1 | tee ../entregas/<RA>/terraform-apply.txt
  ```

### 2. Glue Job (Critério 2 — 25pts)
- **`glue-job-run.txt`**: Status da execução do Glue Job (deve conter `SUCCEEDED`).
  ```bash
  aws glue get-job-run \
    --job-name normaliza-pedidos \
    --run-id <JobRunId> \
    --region us-east-1 > entregas/<RA>/glue-job-run.txt 2>&1
  ```

### 3. Parquet no Gold (Critério 3 — 15pts)
- **`s3-gold-listing.txt`**: Listagem do conteúdo do Bucket_Gold provando o layout `fato_pedidos/data_pedido=YYYY-MM-DD/`.
  ```bash
  aws s3 ls s3://<bucket-gold>/ --recursive > entregas/<RA>/s3-gold-listing.txt 2>&1
  ```

### 4. Consultas Athena (Critério 4 — 15pts)
- **`athena-queries/`**: Prints ou CSVs dos resultados das 4 consultas de referência.
  ```bash
  mkdir -p entregas/<RA>/athena-queries
  # Salvar os resultados de cada consulta executada no console ou via CLI
  ```
  Arquivos esperados:
  - `consulta1_faturamento_categoria.txt`
  - `consulta2_top5_clientes.txt`
  - `consulta3_pedidos_por_dia.txt`
  - `consulta4_integridade.txt`

### 5. DynamoDB (Critério 5 — 10pts)
- **`dynamodb-item.txt`**: Print do item de metadados gravado no DynamoDB.
  ```bash
  aws dynamodb scan --table-name execucoes --region us-east-1 > entregas/<RA>/dynamodb-item.txt 2>&1
  ```

### 6. Limpeza e Custo (Critério 6 — 15pts)
- **`terraform-destroy.txt`**: Saída do `terraform destroy` provando que todos os recursos foram removidos.
  ```bash
  cd infra
  terraform destroy 2>&1 | tee ../entregas/<RA>/terraform-destroy.txt
  ```

### 7. Testes Locais (validação extra)
- **`local-test-results.txt`**: Saída dos testes locais PySpark (pytest no Docker).
  ```bash
  cd local-test
  docker build -t prova-local . && docker run --rm prova-local 2>&1 | tee ../entregas/<RA>/local-test-results.txt
  ```

## Estrutura Final Esperada
```
entregas/<RA>/
├── terraform-apply.txt
├── glue-job-run.txt
├── s3-gold-listing.txt
├── athena-queries/
│   ├── consulta1_faturamento_categoria.txt
│   ├── consulta2_top5_clientes.txt
│   ├── consulta3_pedidos_por_dia.txt
│   └── consulta4_integridade.txt
├── dynamodb-item.txt
├── terraform-destroy.txt
└── local-test-results.txt
```
