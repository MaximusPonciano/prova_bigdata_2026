# Spec 01: Infraestrutura Gold (Terraform)

## Objetivo
Provisionar a camada gold completa na AWS via Terraform, integrada ao `raw.tf` existente, com um único `terraform apply` na pasta `infra/`.

## Recursos Exigidos (a criar em `infra/gold.tf` ou novos arquivos `.tf`)

### 1. Bucket_Gold (S3 — via AWS CLI)
- Criar via `terraform_data` + `local-exec` (AWS CLI), espelhando o padrão de `raw.tf`.
- **Privado**: 4 bloqueios do `public_access_block` habilitados.
- Destrutor: `aws s3 rb s3://<bucket> --force || true`.
- Nome globalmente único (variável `bucket_gold_nome`).

### 2. Upload do Script PySpark para o S3
- Subir `glue-job/normaliza_pedidos.py` para o Bucket_Gold (ou um path dedicado).
- O `script_location` do Glue Job apontará para este path S3.
- Usar `terraform_data` + `local-exec` com `aws s3 cp`.

### 3. AWS Glue Job (`aws_glue_job`)
- `command.name = "glueetl"` (tipo ETL PySpark).
- `command.script_location = "s3://<gold>/scripts/normaliza_pedidos.py"`.
- `role_arn` = ARN da **LabRole** (via `data source` ou `var.labrole_arn`).
- Workers econômicos: `glue_version = "4.0"`, `number_of_workers = 2`, `worker_type = "G.1X"`.
- `default_arguments` com os 4 argumentos do Job: `--RAW_PATH`, `--GOLD_PATH`, `--DDB_TABLE`, `--DATASET_NAME`.
- Tags de custo padronizadas.

### 4. AWS Glue Catalog Database (`aws_glue_catalog_database`)
- Criar uma database no Glue Data Catalog (ex: `prova_bigdata`).
- Registrar as 3 tabelas do modelo dimensional:
  - `fato_pedidos` (com partição `data_pedido`).
  - `dim_cliente` (sem partição).
  - `dim_produto` (sem partição).
- Alternativa: usar um `aws_glue_crawler` com LabRole para descobrir as tabelas automaticamente.

### 5. AWS Athena Workgroup (`aws_athena_workgroup`)
- Nome descritivo (ex: `prova-bigdata-wg`).
- `result_configuration.output_location` apontando para `s3://<gold>/athena-results/`.
- Tags de custo padronizadas.

### 6. AWS DynamoDB Table (`aws_dynamodb_table`)
- Nome: `execucoes` (ou variável).
- `billing_mode = "PAY_PER_REQUEST"` (sem capacidade provisionada).
- `hash_key = "execution_id"` (tipo `S`).
- Tags de custo padronizadas.

## Variáveis Necessárias (a declarar em `variables.tf`)
- `bucket_gold_nome` — nome globalmente único do bucket gold.
- `labrole_arn` — ARN da LabRole do Learner Lab.
- `tags` — mapa de tags padronizadas (Projeto, Disciplina, Ambiente).

## Regras Invioláveis
- **NÃO usar `aws_s3_bucket`** (SCP do Learner Lab nega `GetBucketObjectLockConfiguration`).
- **NÃO criar** `aws_iam_role`, `aws_iam_policy`, `aws_iam_user` ou `aws_iam_group`.
- Provider AWS **fixado em 5.31.0** (`versions.tf` — não alterar).
- Backend **local** (`terraform.tfstate` na própria pasta — já configurado em `versions.tf`).

## Critério de Nota (RUBRICA.md)
- **Critério 1:** Terraform aplica sem erro — **20 pontos**.
- **Critério 6:** Tags, buckets privados, LabRole, destroy — **15 pontos**.
