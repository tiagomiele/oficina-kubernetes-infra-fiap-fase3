# Oficina Kubernetes Infrastructure — Fase 3

Infraestrutura como código do ambiente Kubernetes da oficina mecânica na AWS Academy.

## Responsabilidades

- VPC, subnets, rotas, Internet Gateway e NAT Gateway;
- Amazon EKS e node groups;
- Load Balancer e conectividade privada;
- namespaces, escalabilidade e HPA;
- integração Kubernetes do New Relic;
- outputs de rede consumidos pelos outros repositórios.

Não contém aplicação Spring Boot, Lambda, schema ou provisionamento do RDS.

## Arquitetura

```mermaid
flowchart TB
    APIGW[API Gateway] --> LB[Load Balancer]
    LB --> EKS[Amazon EKS]
    EKS --> Pods[Pods da aplicação]
    HPA[HPA] --> Pods
    EKS --> NR[New Relic nri-bundle]
```

## Tecnologias planejadas

- Terraform e HCP Terraform;
- AWS VPC, EKS e Load Balancer;
- Kubernetes e Helm;
- New Relic `nri-bundle`;
- GitHub Actions.

## Validação

A infraestrutura será implementada na Semana 2. O CI inicial protege a documentação e será ampliado com `terraform fmt`, `validate`, TFLint e análise de segurança.

## Documentação

- [Arquitetura](docs/architecture.md)
- [AWS Academy](docs/aws-academy.md)
- [New Relic no Kubernetes](docs/new-relic.md)
- [Repositórios da solução](docs/repositories.md)

## Contribuição

- mudanças somente por Pull Request;
- `main` representa produção;
- `homolog` representa homologação;
- plan obrigatório antes do apply;
- nenhum segredo ou arquivo de state versionado.
