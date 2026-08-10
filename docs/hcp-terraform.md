# HCP Terraform e execução do Kubernetes

## Workspaces

Crie dois workspaces com execução remota e **Auto apply desativado**:

| Workspace sugerido | Branch | Variável `environment` |
|---|---|---|
| `oficina-kubernetes-homolog` | `homolog` | `homolog` |
| `oficina-kubernetes-production` | `main` | `production` |

A observabilidade New Relic usa dois workspaces adicionais, com **Terraform
Working Directory** igual a `observability/newrelic`:

| Workspace sugerido | Branch | Variável `environment` |
|---|---|---|
| `oficina-newrelic-homolog` | `homolog` | `homolog` |
| `oficina-newrelic-production` | `main` | `production` |

Cada workspace mantém state próprio. Não reutilize o state combinado da Fase 2.

## Variáveis do HCP Terraform

Cadastre como variáveis de ambiente sensíveis e renove a cada sessão do Learner Lab:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```

Cadastre como variáveis Terraform:

```text
aws_region = "us-west-2"
environment = "homolog" ou "production"
lab_role_arn = "arn:aws:iam::<conta>:role/LabRole"
```

As demais variáveis possuem defaults compatíveis com o laboratório. Restrinja `cluster_public_access_cidrs` para o IP público do operador em `/32` quando possível.

Nos workspaces de observabilidade, cadastre `newrelic_account_id`,
`environment`, `cluster_name` e a variável sensível `newrelic_api_key` (ou a
variável de ambiente `NEW_RELIC_API_KEY`). A license key do cluster **não** é
cadastrada aqui: ela existe apenas como secret do GitHub Environment e como
Secret Kubernetes.

## Integração com GitHub Actions

Em cada GitHub Environment (`homolog` e `production`), configure:

- secret `TF_API_TOKEN`;
- variable `TF_CLOUD_ORGANIZATION`;
- variable `TF_WORKSPACE_HOMOLOG`;
- variable `TF_WORKSPACE_PRODUCTION`;
- variable `TF_WORKSPACE_OBSERVABILITY_HOMOLOG`;
- variable `TF_WORKSPACE_OBSERVABILITY_PRODUCTION`.

A lista completa de secrets e variables está em
[Deploy, rollback e troubleshooting](deployment.md).

O workflow **Terraform plan** é manual. Ele seleciona o stack
(`infrastructure` ou `observability`) e o workspace pelo ambiente, e envia o
plan para execução remota no HCP Terraform.

O `workflow_dispatch` só aparece no GitHub Actions depois que o arquivo do workflow existe na branch padrão `main`. No primeiro bootstrap, promova o workflow até `main` ou execute o plan pela CLI com `TF_CLOUD_ORGANIZATION` e `TF_WORKSPACE`; em ambos os casos, mantenha Auto apply desativado.

## Ordem segura

1. inicie a sessão do AWS Academy;
2. confirme `aws sts get-caller-identity` localmente;
3. copie as três credenciais temporárias para o workspace;
4. confirme o ARN atual da `LabRole`;
5. execute **Actions → Terraform plan → Run workflow**, selecionando `homolog`, ou use a CLI no primeiro bootstrap;
6. revise VPC, subnets, NAT Gateway, EKS, node group e outputs;
7. somente depois de aprovação explícita, confirme o apply no HCP Terraform;
8. copie os outputs de rede para o workspace do banco;
9. execute o workflow **Deploy** com `deploy_addons = true` para instalar
   Metrics Server e `nri-bundle`;
10. execute o workflow **Deploy** com `apply_observability = true` para criar
    dashboards e alertas;
11. colete evidências e destrua os recursos ao final.

Nenhum workflow deste repositório executa apply automático por push: o workflow
`Deploy` é manual, exige flags explícitas por etapa e depende da aprovação do
GitHub Environment.
