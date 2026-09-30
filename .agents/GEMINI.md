# Regras Globais do Repositório (Big Data 2026)

> **Estas regras são estritas e invioláveis. Siga-as rigorosamente durante todo o projeto.**

## 1. Regras do AWS Academy Learner Lab (CRÍTICO)
- **NÃO crie** recursos de permissão (IAM) sob nenhuma circunstância. É estritamente proibido criar `aws_iam_role`, `aws_iam_policy`, `aws_iam_user`, ou `aws_iam_group`.
- Sempre que for necessário conceder permissões a um serviço AWS (ex: Glue Job, Crawler), você **DEVE** utilizar a role pré-existente `LabRole`, referenciada por **ARN** (via `data source` ou variável `labrole_arn`).
- A região da AWS **DEVE** ser `us-east-1` em todos os providers, comandos e recursos.
- **Credenciais temporárias:** obtidas no painel "AWS Details" do Learner Lab. NUNCA versionar Access Key, Secret Key ou Session Token. Configurar via variáveis de ambiente ou `~/.aws/credentials`.
- **AWS CLI v2 obrigatório:** os buckets S3 são criados via AWS CLI dentro do `terraform apply` (padrão `terraform_data` + `local-exec`). Sem o CLI instalado e autenticado, o apply falha.


## 3. Restrições de Escopo (Scope Restrictions)
- Você não tem autorização para apagar ou sobrescrever os arquivos de requisitos (`README.md`, `RUBRICA.md`, `CHECKLIST.md`) nem os arquivos entregues prontos pelo professor.
- **NÃO altere:** `infra/raw.tf`, `infra/versions.tf`, `dataset/`, `local-test/`.
- Modifique e crie código **APENAS** em: `infra/gold.tf` (ou novos `.tf` na pasta `infra/`), `infra/variables.tf` (novas variáveis), `infra/terraform.tfvars` e `glue-job/`.
- Entregas do aluno ficam em `entregas/<RA>/`.

## 4. Padrão de Versionamento (Git)
- Todo e qualquer commit deve ser feito utilizando a convenção do **Conventional Commits**: `feat:`, `fix:`, `docs:`, `chore:`, `infra:`.
- Branch no padrão `prova-SEURA` (ex: `prova-2500123`).

## 5. Práticas de DevSecOps (Gestão de Segredos)
- **Tolerância Zero a Vazamentos**: NENHUMA credencial real, token ou senha pode ser colocada de forma estática (hardcoded) no código ou em variáveis do Terraform.
- Não adicione e não comite o arquivo `terraform.tfvars` com dados sensíveis (apenas o `terraform.tfvars.example`).
- **Segurança do Estado IaC**: O arquivo `terraform.tfstate` (e seus backups) é rigorosamente proibido de versionar; atente-se sempre às regras do `.gitignore`.
- Buckets **DEVEM** ser privados com os 4 bloqueios de acesso público habilitados (`BlockPublicAcls`, `IgnorePublicAcls`, `BlockPublicPolicy`, `RestrictPublicBuckets`).

## 6. Padrões Terraform / IaC
- **NÃO use `aws_s3_bucket`**: a SCP da organização do Learner Lab nega `s3:GetBucketObjectLockConfiguration`, que esse recurso sempre lê, causando `AccessDenied`. Crie buckets via `terraform_data` + `local-exec` (AWS CLI), espelhando o padrão de `raw.tf`.
- Provider AWS **fixado em versão 5.31.0** (`versions.tf` — não alterar). Backend local.
- DynamoDB com `billing_mode = PAY_PER_REQUEST` (sem capacidade provisionada, sem custo residual).
- **Tags de custo padronizadas** (`Projeto`, `Disciplina`, `Ambiente`) em **todos** os recursos que suportam tags (Glue Job, Crawler, Athena, DynamoDB).

## 7. Bug Reporting Policy
- Se um comando (como `terraform plan` ou `aws glue start-job-run`) falhar, **NÃO tente adivinhar credenciais** ou ocultar o erro modificando arquivos cegamente.
- Mantenha a evidência da falha e reporte imediatamente informando exatamente a linha com problema.
- NUNCA assuma que ignorar um erro é a solução.

## 8. Completion Checks (Validação de Fim de Tarefa)
- **NUNCA encerre uma tarefa** sem antes rodar o comando mais estreito de validação.
- Para Terraform: execute obrigatoriamente `terraform fmt` para aplicar o padrão visual, seguido de `terraform validate` e `terraform plan`.
- Para PySpark (teste local): execute `docker build -t prova-local . && docker run --rm prova-local` na pasta `local-test/` para validar a normalização.
- Se a validação não puder ser rodada localmente, explique o porquê antes de concluir a execução.

## 9. Copilot Audit Trail (Log Contínuo de IA)
- A prova pode exigir documentação detalhada de como a IA foi utilizada.
- Para automatizar isso: **sempre** que concluir a implementação de uma Spec (01, 02 ou 03) ou realizar uma orientação importante, anexe um log detalhado no arquivo `AI_PROMPT_LOG.md`.
- Formato obrigatório no log: `## [Fase/Spec]`, `**Objetivo/Prompt**:`, `**O que gerou bem**:`, `**Correções Necessárias**:`.
- **Refinamento de Escrita (Implícito)**: Ao transcrever o "Objetivo/Prompt" do usuário para o log, a IA deve **reescrever e polir** a solicitação em tom profissional e corporativo, corrigindo erros de português e elevando o jargão técnico.

## 10. Fluxo de Aprovação Estrita (Strict Proposal Workflow)
- **NÃO implemente código ou avance** sem antes propor um plano detalhado.
- Sempre que houver uma nova etapa ou pedido, a IA deve criar um arquivo `.md` (ex: `PLANO_FASE_X.md`) detalhando a arquitetura, as micro-tarefas e os arquivos que pretende criar/modificar.
- Somente **após a aprovação explícita** do usuário (Líder Técnico), a IA terá permissão para gerar o código real e executar as alterações no ambiente.
