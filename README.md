Introduction

Dans ce tutoriel, nous allons configurer un Raspberry Pi pour formater automatiquement une clé USB lorsqu'elle est branchée. Une interface graphique affichera une barre de progression et jouera une musique une fois le formatage terminé. L'interface sera adaptée à un écran 2.4 pouces (320x240 pixels) et s'affichera en plein écran, et elle sera toujours disponible au démarrage du Raspberry Pi.

1. Prérequis

Matériel
Un Raspberry Pi (modèle avec port USB)
Une clé USB
Un écran 



Installation des dépendances :
```
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip dosfstools mpg321 -y
pip3 install customtkinter --break-system-packages
```


2. Création du Script de Formatage Automatique

Le script auto_format_usb.sh sera responsable du formatage de la clé USB.
Créer le fichier de formatage
```
sudo nano /usr/local/bin/auto_format_usb.sh
```
Ajoutez ce code :
```
#!/bin/bash

LOGFILE="/var/log/usb_format.log"
echo "=== Formatage USB - $(date) ===" >> $LOGFILE

DEVICE=$(lsblk -o NAME,TRAN | grep usb | awk '{print $1}' | head -n 1)
if [ -z "$DEVICE" ]; then
    echo "Aucun périphérique USB détecté !" >> $LOGFILE
    exit 1
fi

DEVICE_PATH="/dev/$DEVICE"
echo "Périphérique détecté : $DEVICE_PATH" >> $LOGFILE

sudo umount ${DEVICE_PATH}* 2>> $LOGFILE || echo "Non monté." >> $LOGFILE

USB_LABEL="Drive"
echo "Formatage de $DEVICE_PATH avec le label $USB_LABEL..." >> $LOGFILE

sudo mkfs.vfat -F 32 -n $USB_LABEL $DEVICE_PATH >> $LOGFILE 2>&1

if [ $? -eq 0 ]; then
    echo "Formatage terminé avec succès pour $DEVICE_PATH !" >> $LOGFILE
else
    echo "Erreur lors du formatage !" >> $LOGFILE
fi

```

Sauvegarde et fermeture : CTRL+O, ENTER, CTRL+X



Rendre le script exécutable :
```
sudo chmod +x /usr/local/bin/auto_format_usb.sh
```


3. Lancer ou arrêter temporairement l'interface graphique

Désactiver temporairement l'interface graphique

Si vous souhaitez désactiver l'interface temporairement, sans la supprimer complètement, utilisez :
```
sudo systemctl stop usb-gui.service
```
Cela arrêtera l'interface jusqu'au prochain redémarrage.

Réactiver l'interface graphique

Si vous voulez la relancer manuellement sans redémarrer le Raspberry Pi, utilisez :
```
sudo systemctl start usb-gui.service
```
Désactiver l'interface au démarrage

Si vous souhaitez empêcher l'interface de démarrer automatiquement au prochain redémarrage :
```
sudo systemctl disable usb-gui.service
```
Réactiver l'interface au démarrage

Si vous voulez qu'elle redémarre automatiquement :
```
sudo systemctl enable usb-gui.service
```
4. Lancer l'interface graphique automatiquement au démarrage

Pour que l'interface de formatage USB démarre automatiquement à chaque démarrage du Raspberry Pi, nous avons créé un service systemd.

Créer le service systemd
```
sudo nano /etc/systemd/system/usb-gui.service
```
Ajoutez ce contenu :
```
[Unit]
Description=Interface Graphique de Formatage USB
After=multi-user.target

[Service]
ExecStart=/usr/bin/python3 /usr/local/bin/usb_format_gui.py
Restart=always
User=(METTRE LE NOM DE VOTRE UTILISATEUR)
Environment=DISPLAY=:0
Environment=SDL_FBDEV=/dev/fb1

[Install]
WantedBy=multi-user.target
```
Sauvegarde et fermeture : CTRL+O, ENTER, CTRL+X

Activez et démarrez le service :
```
sudo systemctl daemon-reload
sudo systemctl enable usb-gui.service
sudo systemctl start usb-gui.service
```
Redémarrez le Raspberry Pi pour tester :

sudo reboot

Installation de l'écran
[si vous voulez le meme que le mien dans la video](https://s.click.aliexpress.com/e/_onk5ohm)
```
sudo rm -rf LCD-show
git clone https://github.com/goodtft/LCD-show.git
chmod -R 755 LCD-show
cd LCD-show/
sudo ./LCD24-3A+-show
```

Après le redémarrage, l'interface se lancera automatiquement sur l'écran 2.4 pouces et restera disponible en permanence. ✅

