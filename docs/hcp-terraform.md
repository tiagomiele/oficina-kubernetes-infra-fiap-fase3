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

O workflow **Terraform plan** executa os stacks `infrastructure` e `observability` automaticamente nos Pull Requests que alteram Terraform. Também pode ser iniciado manualmente por ambiente. Os plans usam `homolog-plan` ou `production-plan`, sem reviewers e sem apply.

Merges em `homolog` iniciam o workflow **Deploy** com infraestrutura, add-ons e observabilidade em um único job sequencial, sem aprovação manual. Merges em `main` usam uma única aprovação no GitHub Environment `production`. O `workflow_dispatch` repete o deploy completo na branch correspondente durante bootstrap ou recuperação. Destroy é manual via Terraform CLI; Auto apply do HCP continua desativado.

## Ordem segura

1. inicie a sessão do AWS Academy e execute o script central uma vez;
2. revise os plans de VPC, subnets, NAT Gateway, EKS, node group e observabilidade no Pull Request;
3. faça merge em `homolog` para executar automaticamente infraestrutura, add-ons e observabilidade;
4. reexecute o script central para propagar os outputs de rede;
5. use `workflow_dispatch` somente para bootstrap ou recuperação;
6. colete evidências e destrua os recursos manualmente quando necessário.

Pull Requests nunca executam apply. O gate é obrigatório somente no merge em `main`, antes do deploy de produção.
