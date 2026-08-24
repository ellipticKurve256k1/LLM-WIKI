```bash
#!/usr/bin/env bash

set -euxo pipefail

apt-get update
apt-get install -y docker.io docker-compose-v2 make

systemctl enable --now docker
usermod -aG docker ubuntu

```