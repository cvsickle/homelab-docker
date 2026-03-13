# homelab-docker
Doco CD configuration for the Docker hosts in my homelab.

# SOPS

This repository uses Mozilla Secret OperationS (SOPS) to encrypt secrets.

To encrypt an env file:
```bash
sops --age=age1v59tukq0cvskn0ww9dwhh9z4ytgj03u599rtzs8xap83jtm8msssc9z8q7 --encrypt secrets.env > sops-secrets.env
```