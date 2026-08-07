# Arquitetura Kubernetes

## Escopo

Este repositório é proprietário da rede compartilhada e do cluster EKS. Seus outputs não sensíveis são consumidos explicitamente pelos repositórios de banco e autenticação.

## Componentes

- VPC em duas zonas de disponibilidade;
- subnets públicas e privadas;
- Internet Gateway e um NAT Gateway compatível com o orçamento do Learner Lab;
- cluster EKS com logs de control plane;
- managed node group privado reutilizando a `LabRole`;
- add-ons VPC CNI, CoreDNS e kube-proxy;
- outputs necessários ao RDS e ao deploy da aplicação;
- preparação para Metrics Server, HPA e New Relic após a criação do cluster.

## Estados

Homologação e produção utilizam estados HCP Terraform separados. O state nunca é armazenado no Git e não reutiliza o state combinado da Fase 2.

## Dependências

A infraestrutura do banco consumirá VPC e subnets privadas. A autenticação serverless consumirá rede e endpoints necessários. A aplicação publicará sua imagem separadamente.
