# HCP Terraform e execução do Kubernetes

## Workspaces

Crie dois workspaces com execução remota e **Auto apply desativado**:

| Workspace sugerido | Branch | Variável `environment` |
|---|---|---|
| `oficina-kubernetes-homolog` | `homolog` | `homolog` |
| `oficina-kubernetes-production` | `main` | `production` |

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

## Integração com GitHub Actions

Em cada GitHub Environment (`homolog` e `production`), configure:

- secret `TF_API_TOKEN`;
- variable `TF_CLOUD_ORGANIZATION`;
- variable `TF_WORKSPACE_HOMOLOG`;
- variable `TF_WORKSPACE_PRODUCTION`.

O workflow **Terraform plan** é manual. Ele seleciona o workspace pelo ambiente e envia o plan para execução remota no HCP Terraform.

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
9. colete evidências e destrua os recursos ao final.

Nenhum workflow deste repositório executa apply automático.
