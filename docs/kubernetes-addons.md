# Add-ons e escalabilidade

O Terraform instala os add-ons gerenciados essenciais:

- Amazon VPC CNI;
- CoreDNS;
- kube-proxy.

O Metrics Server, o HPA da aplicação e o New Relic dependem de acesso funcional ao cluster e serão aplicados após o EKS estar disponível:

1. Metrics Server antes de validar métricas do HPA;
2. Deployment, Service e HPA pelo repositório da aplicação;
3. `nri-bundle` na etapa de observabilidade.

Essa separação evita que a criação inicial do EKS dependa do provider Kubernetes antes de o endpoint existir.
