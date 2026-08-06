# AWS Academy

## Restrições consideradas

- credenciais temporárias;
- reutilização obrigatória da `LabRole` quando IAM estiver bloqueado;
- possível bloqueio de criação de EKS Access Entry;
- recursos destruídos ou indisponíveis após expiração do laboratório;
- necessidade de renovar GitHub Environment e HCP Terraform.

## Estratégia

- validar `aws sts get-caller-identity` no início da pipeline;
- reutilizar roles existentes;
- manter Terraform idempotente e reproduzível;
- separar estados de homologação e produção;
- executar destroy após coleta das evidências;
- não aplicar automaticamente quando a sessão estiver expirada.
