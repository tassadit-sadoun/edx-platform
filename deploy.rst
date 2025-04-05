
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

