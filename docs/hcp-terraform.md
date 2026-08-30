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

Merges em `homolog` e `main` iniciam o workflow **Deploy** com todas as etapas. Cada
job usa o GitHub Environment correspondente e aguarda seu gate. `workflow_dispatch`
permanece para repetir etapas específicas; Auto apply do HCP continua desativado.

## Ordem segura

1. inicie a sessão do AWS Academy e execute o script central uma vez;
2. faça merge na branch do ambiente; o workflow inicia e aguarda aprovação;
3. revise o plan de VPC, subnets, NAT Gateway, EKS, node group e outputs;
4. aprove o GitHub Environment para executar infraestrutura, add-ons e observabilidade;
5. reexecute o script central para propagar os outputs de rede;
6. use `workflow_dispatch` apenas quando precisar repetir uma etapa específica;
7. colete evidências e destrua os recursos ao final.

Pull Requests nunca executam apply; merges iniciam o fluxo, mas o gate do ambiente
continua obrigatório.
