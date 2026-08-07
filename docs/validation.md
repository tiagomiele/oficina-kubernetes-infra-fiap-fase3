# Validação do Terraform Kubernetes

## Validação estática

```bash
terraform fmt -check -recursive
terraform init -backend=false -input=false
terraform validate
```

Quando disponível:

```bash
tflint --recursive
```

O CI também executa Trivy para configurações críticas e Gitleaks.

## Plan remoto

O plan real depende das credenciais temporárias do AWS Academy e do workspace HCP Terraform configurado. Execute o workflow manual `Terraform plan` e confirme:

- uma VPC com DNS habilitado;
- duas subnets públicas e duas privadas em zonas distintas;
- Internet Gateway e um NAT Gateway;
- EKS e managed node group reutilizando somente a `LabRole`;
- nodes nas subnets privadas;
- logs do control plane habilitados;
- nenhuma criação de IAM role ou Access Entry;
- outputs de VPC, subnets e security group do EKS.

O único NAT Gateway é uma decisão de custo para o Learner Lab. Produção real exigiria um NAT por zona ou VPC endpoints equivalentes.

O apply não faz parte da validação estática e exige aprovação separada.
