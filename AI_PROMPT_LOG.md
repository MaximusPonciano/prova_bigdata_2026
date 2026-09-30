# Log Contínuo de IA (Copilot Audit Trail)

## [Fase 1 - Infraestrutura Gold]
**Objetivo/Prompt**: O usuário solicitou a criação da infraestrutura para a camada Gold, Athena e DynamoDB de acordo com as restrições da AWS Academy, contornando a Service Control Policy (SCP) que impede o uso de `aws_s3_bucket`.
**O que gerou bem**: O Terraform foi modularizado com sucesso em `gold.tf`, `glue_dynamo.tf` e `athena.tf`. A limitação do S3 foi superada utilizando a resource `terraform_data` em conjunto com o provisioner `local-exec` e chamadas à AWS CLI. O `terraform fmt` e `validate` foram executados com sucesso.
**Correções Necessárias**: Foi necessário corrigir o script Python para injetar as variáveis da infraestrutura (`bucket_gold_nome`, `bucket_resultados_nome`, `lab_role_arn`) antes que os recursos fossem provisionados de fato no Glue.

## [Fase 2 - Normalização PySpark]
**Objetivo/Prompt**: O usuário solicitou a implementação da lógica ETL de PySpark no arquivo `glue-job/normaliza_pedidos.py` para converter os dados brutos num Esquema Estrela (Fato e Dimensões) tratando dados faltantes.
**O que gerou bem**: A lógica foi implementada priorizando funções puras (sem efeitos colaterais na AWS) no PySpark, agrupando as dimensões e usando a função `coalesce` para tratar strings vazias com a flag `DESCONHECIDO`. A suíte de testes do professor baseada em Docker foi executada localmente gerando 38 testes passados (100% de sucesso).
**Correções Necessárias**: Houve um ajuste na sintaxe do tratamento de erros do `main()` para garantir que a variável `linhas_lidas` persistisse a contagem mesmo quando um erro ocorresse após a leitura do CSV.
