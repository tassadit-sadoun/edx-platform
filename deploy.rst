
Étapes pour configurer un domaine DuckDNS avec EC2 + HTTPS
============================================================

1. Créer ton domaine DuckDNS
-------------------------------
Va sur : https://www.duckdns.org/
Connecte-toi avec ton compte GitHub.
Dans le tableau de bord :
Choisis un nom de sous-domaine (ex. : monopenedx)
Clique sur "add domain"
Tu verras ton domaine : monopenedx.duckdns.org

2. Accéder à ton domaine depuis un navigateur
----------------------------------------------
Teste ::
    http://monopenedx.duckdns.org


3. Fichier Nginx : /etc/nginx/sites-available/openedx
-------------------------------------------------------
Crée-le avec ::
sudo nano /etc/nginx/sites-available/openedx

4. Installer un certificat SSL Let's Encrypt
-------------------------------------------------------
Sur ton instance EC2, exécute :
sudo apt install certbot python3-certbot-nginx -y

Puis ::
sudo certbot --nginx -d monopenedx.duckdns.org


5. Configurer Nginx :
----------------------

Ton fichier de configuration Nginx devrait ressembler à ceci ::

    server {
        listen 80;
        server_name monopenedx.duckdns.org;

        location /studio {
            proxy_pass http://127.0.0.1:8000;  # Port où Open edX tourne
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /admin {
            proxy_pass http://127.0.0.1:8000;  # Port où Open edX tourne
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }



6. Activer la config dans Nginx ::
-------------------------------
    sudo ln -s /etc/nginx/sites-available/openedx /etc/nginx/sites-enabled/

7. Redémarrer Nginx::
-------------------------------
    sudo systemctl restart nginx

Étapes pour déployer Open edX avec Kubernetes sur EC2
============================================================

1. Configuration de base de l’instance EC2
    * Assure-toi d’avoir une instance EC2 avec :
    * Ubuntu 20.04 ou 22.04
    * au moins 2 CPU / 8 Go RAM (16 Go ou + recommandé pour Open edX)
    * Ports ouverts : 80 (HTTP), 443 (HTTPS), 22 (SSH), 3000 (Grafana si tu veux), etc.

2. Installer les dépendances de base::
-------------------------------
    sudo apt update && sudo apt upgrade -y
    sudo apt install -y python3-pip curl git tmux htop ufw

3. Installer Docker et Docker Compose :: 
-------------------------------
    sudo apt install -y docker.io docker-compose
    sudo systemctl enable docker
    sudo usermod -aG docker $USER

4. Installer Kubernetes en local avec K3s (option légère)::
--------------------------------------------------------------

Pour une instance EC2 autonome, le plus simple est d’utiliser K3s (Kubernetes allégé)::
    curl -sfL https://get.k3s.io | sh -

Vérifie que Kubernetes tourne ::
    sudo k3s kubectl get nodes

Alias utile ::
    alias kubectl='sudo k3s kubectl'

5. Installer Helm (gestionnaire de paquets Kubernetes) :: 
--------------------------------------------------------------

    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

6. Installer Tutor ::
--------------------------------------------------------------

    pip install tutor-openedx

7. Configurer Tutor pour Kubernetes
--------------------------------------------------------------
Dis à Tutor que tu veux utiliser Kubernetes et non Docker classique ::
    tutor config save --set K8S_MODE=true

Initialise la config ::
    tutor config save

Ensuite, passe au déploiement Kubernetes :
    tutor k8s quickstart

Cela :
Génère les fichiers Kubernetes (ingress, services, pods)
Crée des volumes persistants
Configure automatiquement les apps
Te demande le superadmin, domaine, email, etc.





