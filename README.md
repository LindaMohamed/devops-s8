# Docker
## 1-1 Why should we run the container with a flag -e to give the environment variables?

Éviter d'écrire les informations sensibles (comme les mots de passe) en dur dans les fichiers, facilitant ainsi leur modification sans reconstruire l'image.

## 1-2 Why do we need a volume to be attached to our postgres container?

Sans volume, les données sont supprimées lorsque le conteneur est détruit. Un volume permet de stocker les données sur l'hôte et de les récupérer même après la suppression du conteneur.

## 1-3 Document your database container essentials: commands and Dockerfile.

**Fichier `Dockerfile` :**

```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY init-scripts/ /docker-entrypoint-initdb.d/

```

 **Commandes Docker associées :**

- **Créer un réseau Docker**
    
    ```bash
    docker network create app-network
    ```
    
- **Construire l’image PostgreSQL**
    
    ```bash
    docker build -t postgres_image .
    ```
    
- **Lancer le conteneur PostgreSQL**
    
    ```bash
    docker run -d --name postgres_container --net=app-network -p 5432:5432 -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -v postgres_data:/var/lib/postgresql/data postgres_image
    ```
    
- **Vérifier si le conteneur est en cours d’exécution**
    
    ```bash
    docker ps
    ```
    
- **Se connecter à la base de données avec psql**
    
    ```bash
    docker exec -it postgres_container psql -U usr -d db
    ```

## 1-4 Why do we need a multistage build? And explain each step of this dockerfile.
Un build multi-stage dans Docker permet d'optimiser la taille de l’image finale tout en gardant un processus de build propre et efficace. Il permet de réduire la taille de l'image en excluant les fihicers inutiles, à la fin il ne reste que le JRE et le code est construit avec une image plus lourdre contenant le JDK.

## 1-5 Why do we need a reverse proxy?
Un reverse proxy est un serveur qui fait le lien entre les clients et les serveurs backend. Il permet une répartition de la charge, un sytème d'authentification centralisée, du chiffrement SSL ainsi que de la journalisation des activités en direcct.

## 1-6 Why is docker-compose so important?
Lorsqu'une application a besoin d'exécuter plusieurs conteneurs, il devient vite complexe de les démarrer spéarement en les faisant communiquer comme il faut. Un docker-compose permet de les orchestrer depuis le même fhicher.

## 1-7 Document docker-compose most important commands. 
```bash
docker-compose up -d
``` 
Démarre tous les services en arrière-plan.

```bash
docker-compose down	
```
Arrête et supprime tous les conteneurs, réseaux et volumes définis.

```bash
docker-compose ps
```
Affiche l'état des services en cours d'exécution.

```bash
docker-compose logs -f <service>
```
Affiche les logs du service en temps réel.

```bash
docker-compose restart backend	
```
Redémarre le service backend sans affecter les autres.

```bash
docker-compose exec backend sh	
```
Ouvre un shell interactif dans le conteneur backend.

```bash
docker-compose build	
```
Reconstruit les images définies dans docker-compose.yml.

## 1-8 Document your docker-compose file.
```yaml
version: '3.7'

services:
  backend:
    image: springboot-api # Utilisation de l'image Docker pour l'API Spring Boot
    ports:
      - "8080:8080" #port 8080 de l'hôte est redirigé vers le port 8080 du conteneur
    container_name: springboot-api    # Nom du conteneur 
    networks:
      - app-network # Connexion du service au réseau app-network
    depends_on:
      - database # Dépendance du service backend vis-à-vis de la base de données

  database:
    image: postgres_image # Utilisation de l'image Docker pour PostgreSQL
    container_name: postgres_container   
    # Variables d'environnement nécessaires à la configuration de PostgreSQL
    environment:
      - POSTGRES_DB=${POSTGRES_DB}  # Nom de la base de données
      - POSTGRES_USER=${POSTGRES_USER}  # Utilisateur
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}  # Mot de passe de l'utilisateur 
    # Connexion du service au réseau app-network
    networks:
      - app-network
    # Volume pour persister les données de la base de données entre les redémarrages du conteneur
    volumes:
      - db-data:/var/lib/postgresql/data

  httpd:
    # Utilisation de l'image pour le serveur Apache
    image: http-server
    container_name: apache-server
    ports: 
      - "80:80" #le port 80 de l'hôte est redirigé vers le port 80 du conteneur
    # Connexion du service HTTPD au réseau app-network
    networks:
      - app-network
    # Dépendance vis-à-vis du service backend (doit démarrer après le backend)
    depends_on: 
      - backend

# Définition des réseaux
networks:
  # Réseau interne permettant la communication entre les services
  app-network:
    external: true  # il doit déjà exister avant d'utiliser ce fichier

# Définition des volumes
volumes:
  # Volume pour persister les données de la base de données PostgreSQL
  db-data:

```

# Github Actions
## 2-1 What are testcontainers?
Les testcontainers sont des bibliothèques Java qui permettent de lancer des conteneurs Docker lors de l'exécution des tests.
## 2-2 Document your Github Actions configurations.
```yaml
# Nom de la pipeline
name: CI devops 2025

# Déclenchement de la pipeline
on:
  push:
    branches:
      - main  # La pipeline s'exécutera lors de pushes sur la branche main
  pull_request:  # Cette ligne permet de déclencher la pipeline lors de pull requests .
  
# Définition des jobs qui seront exécutés dans la pipeline
jobs:
  test-backend:  # Nom du job
    runs-on: ubuntu-22.04  # Le job s'exécutera sur une machine virtuelle Ubuntu 22.04

    steps:  # Liste des étapes du job
     
      # Etape 1 : Récupérer le code depuis le dépôt GitHub
      - uses: actions/checkout@v2.5.0

      # Etape 2 : Configurer Java 21 (Corretto) pour l'environnement
      - name: Set up JDK 21  # Nom de l'étape
        uses: actions/setup-java@v3  # Action GitHub qui installe et configure une version spécifique de Java
        with:
          distribution: 'corretto'  # Utilisation de la distribution Corretto de Java
          java-version: '21'  # Installation de la version 21 de Java

      # Etape 3 : Construire et tester l'application avec Maven
      - name: Build and test with Maven  # Nom de l'étape
        run:  # Commande shell à exécuter
          mvn clean verify  # Commande Maven pour nettoyer, compiler, et exécuter les tests
        working-directory: backend/simple-api-student-main  # Répertoire dans lequel Maven doit exécuter la commande
```
## 2-3 For what purpose do we need to push docker images?
Nous poussons les images Docker sur Docker Hub pour plusieurs raisons :
 - Partage et distribution : Une fois que les images sont publiées sur Docker Hub, elles sont accessibles à toute personne ayant les bonnes autorisations.

- Automatisation du déploiement : Lorsque l'image est disponible sur un registre, elle peut être utilisée pour automatiser le déploiement de l'application dans des environnements de production ou de tests sans nécessiter de re-création de l'image à chaque fois.

- Mise à jour continue : En publiant de nouvelles versions des images, on peut garantir que la dernière version de l'application est toujours disponible pour les environnements qui en ont besoin.

## 2-4 Document your quality gate configuration.
```yaml
# Nom de la pipeline
name: CI devops 2025

# Déclenchement de la pipeline
on:
  push:
    branches:
      - main  # La pipeline s'exécutera lors de pushes sur la branche main
  pull_request:  # Cette ligne permet de déclencher la pipeline lors de pull requests .
  
# Définition des jobs qui seront exécutés dans la pipeline
jobs:
  test-backend:  # Nom du job
    runs-on: ubuntu-22.04  # Le job s'exécutera sur une machine virtuelle Ubuntu 22.04

    steps:  # Liste des étapes du job
     
      # Etape 1 : Récupérer le code depuis le dépôt GitHub
      - uses: actions/checkout@v2.5.0

      # Etape 2 : Configurer Java 21 (Corretto) pour l'environnement
      - name: Set up JDK 21  # Nom de l'étape
        uses: actions/setup-java@v3  # Action GitHub qui installe et configure une version spécifique de Java
        with:
          distribution: 'corretto'  # Utilisation de la distribution Corretto de Java
          java-version: '21'  # Installation de la version 21 de Java

      # Etape 3 : Construire et tester l'application avec Maven
      - name: Build and test with Maven  # Nom de l'étape
         run: mvn -B verify sonar:sonar -Dsonar.projectKey=LindaMohamed_devops-s8 -Dsonar.organization=lindamohamed -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=${{ secrets.SONAR_TOKEN }}
        # Cette commande Maven compile, exécute les tests, et lance l'analyse SonarCloud pour la qualité du code.
        working-directory: backend/simple-api-student-main  # Répertoire où Maven doit exécuter la commande.

  # Job 2 : build-and-push-docker-image, utilisé pour construire et pousser les images Docker.
  build-and-push-docker-image:
    needs: test-backend  # Ce job s'exécute après que le job "test-backend" a été terminé avec succès.
    runs-on: ubuntu-22.04  
    
    steps:  
      # Etape 1 : Récupérer le code source depuis GitHub
      - name: Checkout code
        uses: actions/checkout@v4

      # Etape 2 : Se connecter à DockerHub avec les identifiants sécurisés
      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
        # Cette étape permet de se connecter à DockerHub en utilisant des secrets sécurisés pour l'authentification.

      # Etape 3 : Construire et pousser l'image du backend
      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./backend/simple-api-student-main  # Répertoire contenant le Dockerfile pour le backend.
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api:latest  # Tag pour l'image Docker.
          push: ${{ github.ref == 'refs/heads/main' }}  # Pousser l'image seulement si la branche est `main`.

      # Etape 4 : Construire et pousser l'image de la base de données
      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./databases  # Répertoire contenant le Dockerfile pour la base de données.
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-database:latest  # Tag pour l'image Docker.
          push: ${{ github.ref == 'refs/heads/main' }}  # Pousser l'image seulement si la branche est `main`.

      # Etape 5 : Construire et pousser l'image du serveur HTTP (httpd)
      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          context: ./httpd  # Répertoire contenant le Dockerfile pour le serveur HTTP.
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-httpd:latest  # Tag pour l'image Docker.
          push: ${{ github.ref == 'refs/heads/main' }}  # Pousser l'image seulement si la branche est `main`.
```
# Ansible
## 3-1 Document your inventory and base commands
``setup.yml :``
```yaml
all:
  vars:
    # Définition de l'utilisateur SSH par défaut pour tous les hôtes
    ansible_user: admin

    # Spécifie la clé privée SSH utilisée pour se connecter aux hôtes
    ansible_ssh_private_key_file: id_rsa

  children:
    # Groupe d'hôtes pour l'environnement de production
    prod:
      hosts:
        # Nom de l'hôte de production 
        linda.mohamed.takima.cloud:
```
**Commandes de base :**


```bash 
ansible all -i inventories/setup.yml -m ping
```
Cela permet de tester la connectivité avec tous les hôtes listés dans l'inventaire.

```bash 
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```
Cela donne des détails comme la distribution de l'OS des hôtes.

```bash
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```
Cela permet de désinstaller Apache2 de tous les hôtes.
## 3-2 Document your playbook
Un playbook est un fichier contenant une série d’instructions qu’Ansible exécute sur les hôtes définis. 

### Playbook simple
**Contenu du fichier `playbook.yml`**

```yaml
- hosts: all
  gather_facts: false
  become: true

  tasks:
    - name: Test connection
      ping:
      
```

Ce playbook teste la connexion aux hôtes en envoyant une requête `ping` .

### Playbook docker

**Contenu du fichier `playbook.yml`**

```yaml
- hosts: all
 gather_facts: true
 become: true

 tasks:
   # Install prerequisites for Docker
   - name: Install required packages
     apt:
       name:
         - apt-transport-https
         - ca-certificates
         - curl
         - gnupg
         - lsb-release
         - python3-venv
       state: latest
       update_cache: yes

   # Add Docker’s official GPG key
   - name: Add Docker GPG key
     apt_key:
       url: https://download.docker.com/linux/debian/gpg
       state: present

   # Set up the Docker stable repository
   - name: Add Docker APT repository
     apt_repository:
       repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
       state: present
       update_cache: yes

   # Install Docker
  - name: Install Docker
      apt:
        name: docker-ce
        state: present

   # Install Python3 and pip3
   - name: Install Python3 and pip3
     apt:
       name:
         - python3
         - python3-pip
       state: present

   # Create a virtual environment for Python packages
   - name: Create a virtual environment for Docker SDK
     command: python3 -m venv /opt
       creates: /opt/docker_venv  # Only runs if this directory doesn’t exist

   # Install Docker SDK for Python in the virtual environment
   - name: Install Docker SDK for Python in virtual environment
     command: /opt/docker_venv/bin/pip install docker

   # Ensure Docker is running
   - name: Make sure Docker is running
     service:
       name: docker
       state: started
     tags: docker

```
### Playbook docker avec un rôle

**Création du Rôle Docker :**

```bash
ansible-galaxy init roles/docker
```

Cette commande a généré une structure de fichiers et de dossiers, on aura besoin de cex ddeux là:

- `tasks/` → Contient les tâches exécutées par le rôle
- `handlers/` → Contient les gestionnaires pour gérer les notifications

Dans le fichier `roles/docker/tasks/main.yml`, j’ai défini les tâches  pour installer docker que j’avais de base dans le `playbook.yml`.

```yaml
# Install prerequisites for Docker
- name: Install required packages
	apt:
   name:
     - apt-transport-https
     - ca-certificates
     - curl
     - gnupg
     - lsb-release
     - python3-venv
   state: latest
   update_cache: yes

# Add Docker’s official GPG key
- name: Add Docker GPG key
	apt_key:
   url: https://download.docker.com/linux/debian/gpg
   state: present

# Set up the Docker stable repository
- name: Add Docker APT repository
	apt_repository:
   repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
   state: present
   update_cache: yes

# Install Docker
- name: Install Docker
  apt:
    name: docker-ce
    state: present

# Install Python3 and pip3
- name: Install Python3 and pip3
 apt:
   name:
     - python3
     - python3-pip
   state: present

# Create a virtual environment for Python packages
- name: Create a virtual environment for Docker SDK
	command: python3 -m venv /opt/docker_venv
	args:
   creates: /opt/docker_venv  # Only runs if this directory doesn’t exist

# Install Docker SDK for Python in the virtual environment
- name: Install Docker SDK for Python in virtual environment
	command: /opt/docker_venv/bin/pip install docker

# Ensure Docker is running
- name: Make sure Docker is running
	service:
   name: docker
   state: started
	tags: docker
```

Pour utiliser ce rôle, j’ai modifié `playbook.yml` :

```yaml
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker
```
## 3-3 Document your docker_container tasks configuration.
**Réseau docker:**

Je crée deux réseau docker un entre la base de données et le backend et un entre le backend et le httpd.

**Fichier : `roles/network/tasks/main.yml`**

```yaml
# tasks file for network
- name: Create app-network docker  
	docker_network:    
		name: app-network

- name: Create proxy-network docker  
	docker_network:    
		name: proxy-network
```

**Database**:

Je lance ma base de donnée basée sur mon image que j’ai dans le docker hub.

**Fichier : `roles/database/tasks/main.yml`**

```yaml
- name: Launch the database container  
	docker_container:    name: postgres_container    
	image: linda65/tp-devops-database:latest    
	pull: yes    
	env_file: /home/database.env    
	networks:      
		- name: app-network   
	 volumes:      
		 - db-data:/var/lib/postgresql/data

```

**Backend :**

Je fais pareil pour le backend

**Fichier : `roles/backend/tasks/main.yml`**

```yaml
- name: Launch the backend container  
	docker_container:    
		name: springboot-api    
		image: linda65/tp-devops-simple-api:latest   
		pull: yes    
		networks:      
			- name: app-network      
			- name: proxy-network    
		env_file: /home/backend.env 
```

**HTTPD:**

**Fichier : `roles/httpd/tasks/main.yml`**

```yaml
- name: Launch the HTTPD container  
	docker_container:    
		name: apache-server    
		image: linda65/tp-devops-httpd:latest    
		pull: yes    
		ports:      
			- "80:80"    
		networks:      
			- name: proxy-network
```

**Intégration dans le playbook**

J’ajoute tous les rôles dans le playbook
```yaml
- hosts: all
  gather_facts: true
  become: true

  tasks:
    - name: Copy .env backend
      copy:
        src: backend.env
        dest: /home/backend.env
        mode: '0644'

    - name: Copy .env databse
      copy:
        src: database.env
        dest: /home/database.env
        mode: '0644'

    - name: docker role
      include_role:
        name: docker

    - name: network role
      include_role:
        name: network

    - name: database role
      include_role:
        name: database

    - name: backend role
      include_role:
        name: backend

    - name: httpd role
      include_role:
        name: httpd
```