---
name: deploy-terraform-bigdata
description: >-
  Use esta skill ao lidar com o deploy de infraestrutura da prova de Big Data.
  Cobre o fluxo completo: init, validate, plan, apply, disparo do Glue e destroy.
---

# Deploy e Orquestração do Terraform (Big Data)

Esta skill garante o provisionamento correto da infraestrutura e a obediência às regras do AWS Academy Learner Lab.

## 1. Pré-requisitos
Antes de qualquer operação Terraform, confirme que:
- **Sessão do Learner Lab ativa** e credenciais temporárias exportadas:
  ```bash
  export AWS_ACCESS_KEY_ID="ASIA...."
  export AWS_SECRET_ACCESS_KEY="...."
  export AWS_SESSION_TOKEN="...."
  export AWS_DEFAULT_REGION="us-east-1"
  ```
- **AWS CLI v2** instalado e autenticado (os buckets são criados via CLI no apply).
- **Terraform >= 1.5** instalado.
- Arquivo `terraform.tfvars` criado a partir de `terraform.tfvars.example`:
  ```bash
  cd infra
  cp terraform.tfvars.example terraform.tfvars
  # Editar: regiao, bucket_raw_nome, bucket_gold_nome, labrole_arn, tags
  ```

> ⚠️ As credenciais do Learner Lab **EXPIRAM a cada sessão**. Reconfigure-as antes de rodar Terraform/CLI.

## 2. Recursos da Camada Gold (o que o aluno deve provisionar)
Todos definidos em `infra/gold.tf` (ou novos arquivos `.tf` na pasta `infra/`):

| Recurso | Tipo Terraform | Detalhes |
|---------|---------------|---------|
| Bucket_Gold | `terraform_data` + `local-exec` | Privado, 4 bloqueios, destrutor `aws s3 rb --force` |
| Glue Job | `aws_glue_job` | `command.name = "glueetl"`, `role_arn = LabRole`, `script_location` no S3 |
| Glue Catalog | `aws_glue_catalog_database` | + tabelas ou Crawler com LabRole |
| Athena WG | `aws_athena_workgroup` | `output_location` no S3 |
| DynamoDB | `aws_dynamodb_table` | `PAY_PER_REQUEST`, `hash_key = "execution_id"` |
| Upload Script | `terraform_data` + `local-exec` | Sobe o `.py` para o S3 |

## 3. Fluxo de Execução (um único apply)
```bash
cd infra
terraform init          # Baixa providers e inicializa backend local
terraform validate      # Confere consistência do HCL
terraform plan          # Preview das mudanças
terraform apply         # Cria raw (pronto) + gold (aluno)
```

## 4. Pós-Apply: Disparar e Acompanhar o Glue Job
```bash
# Disparar a execução
aws glue start-job-run \
  --job-name normaliza-pedidos \
  --region us-east-1 \
  --arguments '{"--RAW_PATH":"s3://<raw>/pedidos/","--GOLD_PATH":"s3://<gold>/","--DDB_TABLE":"execucoes","--DATASET_NAME":"pedidos_desnormalizado"}'

# Acompanhar status (o comando retorna um JobRunId)
aws glue get-job-run \
  --job-name normaliza-pedidos \
  --run-id <JobRunId> \
  --region us-east-1 \
  --query 'JobRun.JobRunState'
```
O status evolui por: `STARTING → RUNNING → SUCCEEDED` (ou `FAILED`).

## 5. Pós-Job: Consultas Athena
1. Confirmar que as tabelas estão registradas no Glue Data Catalog.
2. Executar as 4 consultas de referência no Athena (Seção 8 do README).
3. Conferir o item de metadados no DynamoDB.

## 6. Limpeza Obrigatória (CRÍTICO)
```bash
cd infra
terraform destroy       # Remove tudo (buckets somem pelos provisioners de destroy)
```

> ⚠️ O Learner Lab tem **orçamento e tempo de sessão limitados**. Execute `terraform destroy` antes de encerrar a sessão para evitar consumo residual.

## Hard Rules
- **NUNCA** provisionar Roles (`aws_iam_role`) — penalidade grave nas regras do Learner Lab.
- **NUNCA** usar `aws_s3_bucket` — usar `terraform_data` + `local-exec` (AWS CLI).
- **SEMPRE** rodar `terraform validate` e `terraform plan` antes de propor o `terraform apply`.
- **SEMPRE** guardar evidências (logs do apply, destroy, Glue Job) para a entrega.
