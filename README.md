# Doco-CD Deployment

This repository is a Doco-CD deployment for the Docker hosts in my homelab. If you're reading this, and you're not the owner of this homelab, you really shouldn't try to use this repository directly. Feel free to reference the deployment strategy and specifics, but don't try to use them unless you know what you're doing.

- GitHub: [https://github.com/cvsickle/homelab-docker](https://github.com/cvsickle/homelab-docker)
- Codeberg: [https://codeberg.org/cvsickle/homelab-docker](https://codeberg.org/cvsickle/homelab-docker)
- Forgejo (Mirror): [https://git.cvsickle.com/cvsickle/homelab-docker](https://git.cvsickle.com/cvsickle/homelab-docker)

# SOPS

This repository uses Mozilla Secret OperationS (SOPS) to encrypt secrets.

To encrypt an env file:
```bash
sops --age=age1v59tukq0cvskn0ww9dwhh9z4ytgj03u599rtzs8xap83jtm8msssc9z8q7 --encrypt secrets.env > sops-secrets.env
```