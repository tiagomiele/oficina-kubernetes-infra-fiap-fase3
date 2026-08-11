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

Não copie credenciais ou o ARN da `LabRole` manualmente. Execute no repositório do backend:

```powershell
.\scripts\configure-environment.ps1 -Environment homolog
```

O script cria o Variable Set compartilhado `aws-academy-credentials`, associa homologação e produção, remove credenciais diretas conflitantes e mantém apenas `environment` por workspace. Região e versão do EKS usam defaults; o ARN da `LabRole` é derivado da conta autenticada com `aws_caller_identity`.

Restrinja `cluster_public_access_cidrs` para o IP público do operador em `/32` quando possível.

Para os workspaces de observabilidade, execute o mesmo script com `-ConfigureNewRelic`. A License key permanece somente no GitHub Environment e no Secret Kubernetes.

## Integração com GitHub Actions

O script central cria e atualiza os GitHub Environments `homolog` e `production`, incluindo token HCP, credenciais AWS, workspaces e nomes de cluster. A lista completa está em [Deploy, rollback e troubleshooting](deployment.md).

O workflow **Terraform plan** é manual. Ele seleciona o stack
(`infrastructure` ou `observability`) e o workspace pelo ambiente, e envia o
plan para execução remota no HCP Terraform.

O `workflow_dispatch` só aparece no GitHub Actions depois que o arquivo do workflow existe na branch padrão `main`. No primeiro bootstrap, promova o workflow até `main` ou execute o plan pela CLI com `TF_CLOUD_ORGANIZATION` e `TF_WORKSPACE`; em ambos os casos, mantenha Auto apply desativado.

## Ordem segura

1. inicie a sessão do AWS Academy e execute o script central uma vez;
2. execute **Actions → Terraform plan → Run workflow**, selecionando o ambiente, ou use a CLI no primeiro bootstrap;
3. revise VPC, subnets, NAT Gateway, EKS, node group e outputs;
4. somente depois de aprovação explícita, confirme o apply no HCP Terraform;
5. reexecute o script central para propagar os outputs de rede;
6. execute o workflow **Deploy** com `deploy_addons = true` para instalar Metrics Server e `nri-bundle`;
7. execute o workflow **Deploy** com `apply_observability = true` para criar dashboards e alertas;
8. colete evidências e destrua os recursos ao final.

Nenhum workflow deste repositório executa apply automático por push: o workflow
`Deploy` é manual, exige flags explícitas por etapa e depende da aprovação do
GitHub Environment.
