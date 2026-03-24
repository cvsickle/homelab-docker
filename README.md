# Doco-CD Deployment

This repository is a [Doco-CD](https://github.com/kimdre/doco-cd) deployment for the Docker hosts in my homelab. If you're reading this, and you're not the owner of this homelab, you really shouldn't try to use this repository directly. Feel free to reference the deployment strategy and specifics, but don't try to use them unless you know what you're doing.

- GitHub: [https://github.com/cvsickle/homelab-docker](https://github.com/cvsickle/homelab-docker)
- Codeberg: [https://codeberg.org/cvsickle/homelab-docker](https://codeberg.org/cvsickle/homelab-docker)
- Forgejo (Mirror): [https://git.cvsickle.com/cvsickle/homelab-docker](https://git.cvsickle.com/cvsickle/homelab-docker)

# SOPS

This repository uses Mozilla Secret OperationS (SOPS) to encrypt secrets.

To encrypt an env file:
```bash
sops --age=age1v59tukq0cvskn0ww9dwhh9z4ytgj03u599rtzs8xap83jtm8msssc9z8q7 --encrypt secrets.env > sops-secrets.env
```

# Bootstrap

To deploy the apps to a Docker host, create a folder on that host with the following structure:

```bash
 └── docker
     └── doco-cd
         ├── data                # Data folder for repo contents.
         ├── age.agekey          # Age private key for SOPS decryption.
         ├── apprise.txt         # Apprise notification URLs. (optional)
         ├── docker-compose.yaml # Compose file for doco-cd.
         ├── github.txt          # Text file with GitHub token.
         └── webhook.txt         # Text file with webhook secret.
```

## Docker Compose

Use this template file to define the doco-cd deployment on the host. The `<host>` needs to match a `.doco-cd.<host>.yaml` file in the root of the repository.

```yaml
x-poll-config: &poll-config
  POLL_CONFIG: |
    - url: https://github.com/cvsickle/homelab-docker.git
      reference: main
      target: <host>
      interval: 180

services:
  app:
    container_name: doco-cd
    image: ghcr.io/kimdre/doco-cd:latest
    ports:
      - "8888:80"   # Webhook endpoint
      - "9128:9120" # Prometheus metrics
    environment:
      TZ: America/New_York
      GIT_ACCESS_TOKEN_FILE: /run/secrets/github_token
      WEBHOOK_SECRET_FILE: /run/secrets/webhook_secret
      <<: *poll-config
      SOPS_AGE_KEY_FILE: /run/secrets/sops_age_key
      APPRISE_API_URL: http://apprise:8000/notify # Optional
      APPRISE_NOTIFY_LEVEL: success
      APPRISE_NOTIFY_URLS_FILE: /run/secrets/apprise_urls
    secrets:
      - sops_age_key
      - github_token
      - webhook_secret
      - apprise_urls
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
    labels:
      - "autoheal=true"                             # Optional if using willfarrell/autoheal
      - "com.centurylinklabs.watchtower.scope=true" # Optional if using nickfedor/watchtower
    restart: unless-stopped
    depends_on:
      apprise:
        condition: service_healthy
    depends_on:
      - apprise # Optional
    healthcheck:
      test: [ "CMD", "/doco-cd", "healthcheck" ]
      start_period: 15s
      interval: 30s
      timeout: 5s
      retries: 3
  
  # Optional if using notification
  apprise:
    image: caronc/apprise:latest
    container_name: apprise
    ports:
      - "8889:8000"
    environment:
      TZ: America/New_York
      APPRISE_WORKER_COUNT: 1
    volumes:
      - apprise_config:/config
      - apprise_plugin:/plugin
      - apprise_attach:/attach
    labels:
      - "autoheal=true"                             # Optional if using willfarrell/autoheal
      - "com.centurylinklabs.watchtower.scope=true" # Optional if using nickfedor/watchtower
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/status"]
      start_period: 10s
      interval: 15s
      timeout: 5s
      retries: 3

secrets:
  sops_age_key:
    file: age.agekey
  github_token:
    file: github.txt
  webhook_secret:
    file: webhook.txt
  apprise_urls:
    file: apprise.txt # Optional

# Optional if using notification
volumes:
  apprise_config:
  apprise_plugin:
  apprise_attach:
```

## Apprise 

See [Notifications](https://github.com/kimdre/doco-cd/wiki/Notifications).

The `apprise.txt` file will need setup with instructions to send notfications and the compose file needs to reference this file, as indicated above. See the [Apprise Documentaiton](https://github.com/caronc/apprise/wiki#notification-services) for more information.