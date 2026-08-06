# Arquitetura Kubernetes

## Escopo

Este repositório será proprietário da rede compartilhada e do cluster EKS. Seus outputs não sensíveis serão consumidos pelos repositórios de banco e autenticação.

## Componentes

- VPC em duas zonas de disponibilidade;
- subnets públicas e privadas;
- NAT Gateway e Internet Gateway;
- cluster EKS e managed node group;
- Load Balancer para a aplicação;
- HPA por CPU e memória;
- Metrics Server;
- Helm release do New Relic.

## Estados

Homologação e produção terão estados HCP Terraform separados. O state nunca será armazenado no Git.

## Dependências

A infraestrutura do banco consumirá VPC e subnets privadas. A autenticação serverless consumirá rede e endpoints necessários. A aplicação publicará sua imagem separadamente.
