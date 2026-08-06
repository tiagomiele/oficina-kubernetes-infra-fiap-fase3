# New Relic no Kubernetes

A integração será instalada com o Helm chart oficial `nri-bundle`.

## Telemetria

- nodes, namespaces, deployments, pods e containers;
- CPU, memória, reinícios e réplicas;
- eventos Kubernetes;
- logs JSON da aplicação;
- metadados Kubernetes associados aos traces APM.

## Segurança

A license key será fornecida por secret e não aparecerá em `values.yaml`, Terraform state público ou logs de pipeline.

## Alertas planejados

- deployment sem réplicas disponíveis;
- reinícios repetidos;
- CPU e memória próximas dos limites;
- HPA no máximo de réplicas;
- ausência de telemetria do cluster.
