# Informations

1. LTSP version : 23.02-1+deb12u1
2. Debian 12

# Script-Station-Internet-LTSP
📄 - Ce script permet d'effectuer une mise à jour de l'image LTSP. Il vérifie d'abord si un fichier flag existe pour éviter des exécutions multiples, puis lance une synchronisation avec un serveur via ```rsync```, en excluant certains dossiers. Après la mise à jour, le système redémarre automatiquement pour appliquer les modifications.

**📌 - Ce script doit être ajouté et exécuté au démarrage de la session de l'utilisateur.**

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
