# Informations

1. LTSP version : 23.02-1+deb12u1
2. Debian 12

# Description du script 
**📄 - Ce script assure la mise à jour automatisée de l'image LTSP utilisée par les postes clients d’un environnement en réseau. Pour garantir la sécurité et l'intégrité du processus, il commence par vérifier l’existence d’un fichier témoin (flag) permettant d’éviter une exécution multiple simultanée du script.

S’il n’est pas déjà en cours d’exécution, le script procède à une synchronisation de l’image système à l’aide de la commande rsync, en se connectant à un serveur distant. Certains dossiers critiques ou temporaires sont explicitement exclus de la synchronisation afin de préserver la stabilité et la cohérence de l’image.

Une fois la mise à jour terminée, un redémarrage du système est déclenché automatiquement afin de garantir que tous les changements prennent effet dès le prochain démarrage des clients LTSP.**

**📌 - Ce script doit être ajouté et exécuté au démarrage de la session de l'utilisateur.**

**🐧​ - Script Bash :**
```bash
#!/bin/bash

# Chemin du fichier flag
flag_file="/home/bpx/tags/ffexecuted.flag"

# Si le flag existe déjà → on quitte directement
if [ -f "$flag_file" ]; then
    exit 0
fi

sleep 5

# Si on est pas encore dans un vrai terminal, alors ouvrir xfce4-terminal
if [ "$TERM" = "dumb" ] || [ -z "$TERM" ]; then
    xfce4-terminal --hold --command="$0"
    exit 0
fi

# === Ici, on est dans une vraie fenêtre de terminal ===


echo "#######################################################"
echo "######                                           ######"
echo "####            Mise à jour en cours ...           ####"
echo "######                                           ######"
echo "#######################################################"

touch "$flag_file"
sync  # Force l'écriture sur le disque

# Attendre 5 seconds
sleep 5

clear

echo "#######################################################"
echo "######                                           ######"
echo "####    Synchronisation au serveur en cours ...    ####"
echo "######                                           ######"
echo "#######################################################"
echo ""

# Synchronisation des fichiers
rsync -av --progress --delete-after \
    --exclude='*/tags/' \
    --exclude='*/Bureau/' \
    --exclude='*/Images/' \
    --exclude='*/Documents/' \
    --exclude='*/Téléchargements/' \
    --exclude='*/Vidéos/' \
    --exclude='*/Musique/' \
    --exclude='*/.cache/' \
    --exclude='*/.thunderbird/' \
    --exclude='*/.mozilla/' \
    /etc/bpx /home/

# Attendre 5 seconds
sleep 2

clear

echo "######################################################"
echo "######                                          ######"
echo "####            Mise à jour terminé !             ####"
echo "######                                          ######"
echo "######################################################"
echo ""

for i in {10..1}
do
    echo -ne "\rRedémarrage dans $i secondes..."
    sleep 1
done

# Redémarrage pour permettre la fermeture de session
systemctl reboot
```
