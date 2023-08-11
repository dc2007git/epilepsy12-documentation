---
title: Deployment
authors: Dr Marcus Baw
---

Adds a command to make OS updates quicker and more consistent
```bash
sudo echo "alias doupdates='sudo apt update && sudo apt dist-upgrade && sudo apt autoremove && sudo apt autoclean'" >> /etc/profile;source /etc/profile
```
doupdates
ssh-import-id gh:eatyourpeas
sudo git clone --single-branch --branch development https://github.com/rcpch/rcpch-audit-engine.git /var/rcpch-audit-engine
cd rcpch-audit-engine/
curl -fsSL https://get.docker.com -o get-docker.sh | sudo sh get-docker.sh


