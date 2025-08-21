# ESP32 Roomba

Pilotez un iRobot Roomba depuis un ESP32 : le firmware parle au robot par son port série avec l'interface iRobot® Create® 2 Open Interface, et vous le conduisez, lisez ses capteurs et regardez sa caméra depuis une page web en WiFi.

## Fonctionnalités

- Contrôler le Roomba avec l'ESP32 en WiFi via websocket
- Récupérer les données des capteurs du Roomba
- Contrôler le Roomba depuis une interface web
- Diffusion vidéo depuis la caméra de l'ESP32

## Prérequis

- Un ESP32 (j'ai utilisé un ESP32-CAM OV5640 REV1.2.1)
- Un Roomba compatible (j'ai utilisé un Roomba 564)
- Un régulateur de tension (pour passer de la tension non régulée de la batterie du Roomba à 5 V pour l'ESP32)
- PlatformIO

## Câblage

![Brochage du Roomba](images/roomba-pinout.jpg)

- Reliez BRC du Roomba (broche 5) à l'ESP32 (broche 19)
- Reliez RXD du Roomba (broche 3) à l'ESP32 (broche 16)
- Reliez TXD du Roomba (broche 4) à l'ESP32 (broche 17)
- Reliez GND du Roomba (broche 6 ou 7) au GND d'entrée du régulateur, et son GND de sortie au GND de l'ESP32
- Reliez Vpwr du Roomba (broche 1 ou 2) au Vin d'entrée du régulateur, et son Vout de sortie au 5 V de l'ESP32

Pour la vidéo, utilisez une carte ESP32 avec support caméra (OV2640 ou OV5640 par exemple). Vous pouvez souder des fils directement sous le port série du Roomba et fixer l'ESP32 sous l'avant de la carte mère du Roomba. Reliez la caméra à l'ESP32 avec une longue nappe. Pour installer la caméra, fixez-la à l'avant du Roomba derrière le pare-chocs, et percez un petit trou dans la partie basse du pare-chocs pour que l'objectif puisse filmer.

## Configuration

Créez un fichier config.h dans le dossier src et ajoutez votre SSID et votre mot de passe WiFi :

```cpp
#ifndef CONFIG_H
#define CONFIG_H

const char* ssid = "YourSSID";
const char* password = "YourPassword";

#endif // CONFIG_H
```

## Installation

Utilisez PlatformIO dans Visual Studio Code pour installer les bibliothèques nécessaires et téléverser le code sur votre ESP32.

1. **Première configuration**
    - Sélectionnez votre ESP32 et mettez à jour le fichier `platformio.ini` si besoin

2. **Compiler le projet**
    - Cliquez sur la coche (**Build**) dans la barre d'outils PlatformIO ou lancez `PlatformIO: Build` depuis la palette de commandes.

3. **Construire l'image du système de fichiers**
    - Dans la barre latérale PlatformIO, dépliez **Project Tasks** sous votre projet et sélectionnez votre plateforme ESP32. Cliquez sur **Build Filesystem Image** pour générer l'image ou lancez `PlatformIO: Build Filesystem Image` depuis la palette de commandes.

4. **Téléverser le firmware**
    - Cliquez sur la flèche vers la droite (**Upload**) dans la barre d'outils PlatformIO ou lancez `PlatformIO: Upload`.

5. **Téléverser l'image du système de fichiers**
    - Dans la barre latérale PlatformIO, sous **Project Tasks**, sélectionnez votre plateforme ESP32 et cliquez sur **Upload Filesystem Image** pour envoyer l'image générée ou lancez `PlatformIO: Upload Filesystem Image` depuis la palette de commandes.

6. **Surveiller la sortie série (optionnel)**
    - Cliquez sur l'icône de prise (**Monitor**) ou lancez `PlatformIO: Monitor` pour voir les logs de l'ESP32.

Votre ESP32 devrait maintenant faire tourner le firmware de contrôle du Roomba avec l'image de système de fichiers requise.

## Utilisation

Connectez-vous au réseau WiFi de l'ESP32, ouvrez un navigateur et allez sur `http://<ADRESSE_IP_ESP32>` pour accéder à l'interface de contrôle du Roomba.

Appuyez sur le bouton Connect pour établir la connexion au websocket de l'ESP32 et au flux de la caméra. Appuyez ensuite sur le bouton Wakeup pour réveiller le Roomba.

## Configuration de reverse proxy (optionnel)

Si vous voulez accéder à l'interface de contrôle du Roomba à travers un reverse proxy, vous pouvez utiliser Nginx ou Apache. Voici un exemple simple de configuration Nginx :

```nginx
server {
    listen 80;

    location /esp32-roomba/ {
        auth_basic "Restricted Access"; auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://<ESP32_IP_ADDRESS>:80/;
        proxy_set_header Host $host; proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade"; proxy_set_header X-Real-IP $remote_addr;
        proxy_buffering off; proxy_read_timeout 86400s; proxy_http_version 1.1;
        proxy_hide_header X-ESP32;
    }

    location /esp32-roomba/ws {
        auth_basic "Restricted Access"; auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://<ESP32_IP_ADDRESS>:81/;
        proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade";
        proxy_set_header Host $host; proxy_buffering off; proxy_read_timeout 86400s;
        proxy_http_version 1.1;
    }
}
```
