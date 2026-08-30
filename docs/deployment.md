# Deploy, rollback e troubleshooting

## Ordem segura

1. iniciar a sessão do AWS Academy e executar o script central do backend;
2. confirmar que `aws sts get-caller-identity` foi validado pelo script;
3. merge em `homolog` ou `main`, que inicia o plan e aguarda aprovação do ambiente;
4. **Deploy** automático da infraestrutura (rede, EKS, node group e add-ons gerenciados);
5. **Deploy** automático do Metrics Server e `nri-bundle`;
6. deploy da aplicação pelo repositório do backend (Deployment, Service, HPA);
7. **Deploy** automático dos dashboards, política e condições no New Relic;
8. preencher `health_check_url` e habilitar o monitor sintético;
9. coletar evidências e executar o destroy ao final da sessão.

O workflow `Deploy` respeita essa ordem: `guard → infrastructure → addons →
observability`. Push em `homolog` ou `main` habilita todas as etapas; o GitHub
Environment mantém o gate humano. `workflow_dispatch` permite repetir etapas específicas.

## Ambientes

| Ambiente | Branch permitida | GitHub Environment | Workspaces HCP |
|---|---|---|---|
| homolog | `homolog` | `homolog` | `TF_WORKSPACE_HOMOLOG`, `TF_WORKSPACE_OBSERVABILITY_HOMOLOG` |
| production | `main` | `production` | `TF_WORKSPACE_PRODUCTION`, `TF_WORKSPACE_OBSERVABILITY_PRODUCTION` |

O job `guard` recusa a execução quando a branch não corresponde ao ambiente.
Configure *Required reviewers* em ambos os GitHub Environments; sem isso não
existe gate humano antes do apply.

## Secrets e variables por GitHub Environment

Todos os itens abaixo são sincronizados pelo script central. Não copie valores manualmente.

Secrets:

| Nome | Uso |
|---|---|
| `TF_API_TOKEN` | autenticação no HCP Terraform |
| `AWS_ACCESS_KEY_ID` | sessão temporária do Learner Lab |
| `AWS_SECRET_ACCESS_KEY` | sessão temporária do Learner Lab |
| `AWS_SESSION_TOKEN` | sessão temporária do Learner Lab |
| `NEW_RELIC_LICENSE_KEY` | criação do Secret Kubernetes `newrelic-license` |
| `NEW_RELIC_API_KEY` | provider Terraform do New Relic (User key `NRAK-...`) |

Variables:

| Nome | Uso |
|---|---|
| `TF_CLOUD_ORGANIZATION` | organização HCP Terraform |
| `TF_WORKSPACE_HOMOLOG` / `TF_WORKSPACE_PRODUCTION` | workspaces de infraestrutura |
| `TF_WORKSPACE_OBSERVABILITY_HOMOLOG` / `TF_WORKSPACE_OBSERVABILITY_PRODUCTION` | workspaces de observabilidade |
| `NEW_RELIC_ACCOUNT_ID` | account id numérico |
| `AWS_REGION` | opcional, padrão `us-west-2` |
| `CLUSTER_NAME` | opcional, padrão `oficina-<ambiente>` |
| `APP_NAMESPACE` | opcional, padrão `oficina` |
| `HEALTH_CHECK_URL` | URL do healthcheck para o monitor sintético |
| `SYNTHETIC_MONITOR_ENABLED` | `true` somente após a URL existir |

## Secret Kubernetes da license key

Criado no momento do deploy pelo script `scripts/deploy-cluster-addons.sh`:

| Item | Valor |
|---|---|
| Namespace | `newrelic` |
| Secret | `newrelic-license` |
| Chave | `licenseKey` |
| Referência no chart | `global.customSecretName` e `global.customSecretLicenseKey` |

O comando usa `kubectl create secret --dry-run=client -o yaml | kubectl apply
-f -`, portanto é idempotente e não imprime o conteúdo. A chave nunca aparece
em `values.yaml`, em state do Terraform, em logs ou no resumo do workflow.

## Rollback

| Situação | Ação |
|---|---|
| `nri-bundle` com problema | `helm rollback newrelic-bundle -n newrelic` ou `helm uninstall newrelic-bundle -n newrelic` |
| Metrics Server com problema | `helm rollback metrics-server -n kube-system` |
| Dashboards/alertas indesejados | `terraform apply` com `observability_enabled = false` no workspace de observabilidade |
| Monitor sintético | `synthetic_monitor_enabled = false` e novo apply |
| Infraestrutura | `terraform destroy` no workspace de infraestrutura, após o destroy do banco |

O rollback dos add-ons não remove a aplicação nem o cluster. Remover o cluster
sem antes remover o RDS quebra a referência do security group.

## Troubleshooting

| Sintoma | Causa provável | Ação |
|---|---|---|
| `ExpiredToken` / `ExpiredTokenException` | sessão do Learner Lab encerrada | reiniciar o lab e reexecutar o script central uma vez |
| `InvalidClientTokenId` | credenciais trocadas parcialmente | atualizar os três valores juntos |
| `error: You must be logged in to the server (Unauthorized)` | kubeconfig de outra sessão ou usuário sem acesso ao cluster | reexecutar `aws eks update-kubeconfig` com a sessão atual |
| Pods do `newrelic` em `CrashLoopBackOff` | license key inválida | recriar o Secret com a chave correta e reexecutar o deploy |
| `kubectl top` sem dados | Metrics Server ainda não pronto | aguardar o rollout e revalidar; HPA depende dele |
| HPA com `<unknown>` | ausência de métricas ou de `resources.requests` | validar Metrics Server e requests do Deployment |
| Nenhum dado de Kubernetes no New Relic | `global.cluster` divergente | conferir `CLUSTER_NAME` e o valor de `cluster_name` no workspace de observabilidade |
| Logs duplicados no New Relic | encaminhamento de logs também no agente Java | manter `NEW_RELIC_APPLICATION_LOGGING_FORWARDING_ENABLED=false` na aplicação |
| Condição de alerta sem dados | nome de evento de negócio divergente | alinhar os eventos publicados pela aplicação ao mapa NRQL |

## Controle de custo

- um único NAT Gateway por VPC;
- nodes `t3.medium` com `desired = 2` e `max = 3`;
- `lowDataMode: true` em todos os componentes do `nri-bundle`;
- Prometheus agent restrito a alvos anotados e `scrape_interval` de 60s em
  homologação;
- `nri-prometheus`, Pixie, eBPF e operators desativados;
- retenção de 7 dias nos logs de control plane;
- destroy obrigatório ao final de cada sessão do laboratório.

## Checklist de validação geral

- [ ] `terraform fmt -check -recursive` nas duas configurações;
- [ ] `terraform init -backend=false -lockfile=readonly` e `terraform validate`;
- [ ] TFLint e Trivy sem achados críticos;
- [ ] `scripts/lint-cluster-addons.sh` renderizando os dois ambientes;
- [ ] `yamllint` e `shellcheck` aprovados;
- [ ] CI aprovado no Pull Request;
- [ ] plan revisado antes de qualquer apply;
- [ ] `aws sts get-caller-identity` válido antes do deploy;
- [ ] rollout do Metrics Server e do `nri-bundle` confirmado;
- [ ] pods, HPA e namespace `newrelic` reportados sem exposição de segredos;
- [ ] dashboards e condições de alerta presentes no New Relic;
- [ ] monitor sintético habilitado apenas depois da URL pública existir;
- [ ] nenhum segredo, state ou credencial versionado.
