# TP DevOps — Docker, CI/CD et Ansible

Projet réalisé dans le cadre du module DevOps. L'objectif était de déployer une application web complète en utilisant Docker, GitHub Actions et Ansible.

## L'application

L'application est composée de 4 services :

- **simplestudentapi** — API backend Spring Boot (Java 21)
- **postgrestp1** — Base de données PostgreSQL
- **front** — Interface Vue.js servie par Nginx
- **httpd** — Reverse proxy Apache, seul point d'entrée sur le port 80

```
Browser → Apache (port 80)
           ├── /api/  → simplestudentapi:8080
           └── /      → front:80
```

## Structure du projet

```
├── .github/workflows/
│   ├── main.yml              # CI : tests Maven + analyse SonarCloud
│   └── build-and-push.yml   # CD : build des images + déploiement Ansible
├── ansible/
│   ├── inventories/setup.yml
│   ├── playbook.yml
│   ├── group_vars/all/vault.yml  # variables chiffrées (Ansible Vault)
│   └── roles/
│       ├── docker/           # installation de Docker
│       ├── network/          # création du réseau app-network
│       ├── database/         # lancement de PostgreSQL
│       ├── app/              # lancement du backend
│       ├── front/            # lancement du front
│       └── proxy/            # lancement d'Apache
├── simple-api/               # Backend Spring Boot
├── database/                 # Image PostgreSQL avec scripts SQL
├── front/                    # Frontend Vue.js
└── http-server/              # Configuration Apache
```

## Pipeline CI/CD

À chaque push sur `main` :

1. Les tests Maven sont lancés avec Testcontainers
2. Le code est analysé par SonarCloud
3. Si tout passe, les 4 images Docker sont buildées et publiées sur Docker Hub
4. Ansible déploie automatiquement l'application sur le serveur

## Images Docker Hub

- `ky0u/tp-devops-simple-api:latest`
- `ky0u/tp-devops-database:latest`
- `ky0u/tp-devops-front:latest`
- `ky0u/tp-devops-httpd:latest`

## Déploiement manuel

Prérequis : Ansible installé, clé SSH configurée, secrets dans un fichier vault.

```bash
ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml --ask-vault-pass
```

## Secrets GitHub requis

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Nom d'utilisateur Docker Hub |
| `DOCKERHUB_TOKEN` | Token Docker Hub |
| `SSH_PRIVATE_KEY` | Clé privée SSH pour accéder au serveur |
| `SERVER_IP` | IP du serveur de production |
| `VAULT_PASSWORD` | Mot de passe Ansible Vault |
| `SONAR_TOKEN` | Token SonarCloud |
