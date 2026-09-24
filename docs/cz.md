# Documentation : Samba, NFS et Clonezilla

**Auteur :** Loris.R  
**Module :** B2  
**Sujet :** Solution de partage réseau (Samba, NFS) et de déploiement d'images (Clonezilla)

---

## 1. Samba (SMB/CIFS)

!!! info "Présentation & Cas d'usage"
    **Samba** est la réimplémentation libre du protocole réseau SMB/CIFS. Il permet le partage de fichiers et d'imprimantes entre des systèmes hétérogènes (**Linux et Windows**). Il peut également être configuré comme contrôleur de domaine (Active Directory).

### Installation et Configuration (Debian)

```bash
# Installation des paquets nécessaires
apt update && apt install -y samba samba-common-bin
```

La configuration principale s'effectue dans le fichier `/etc/samba/smb.conf`.

[image de la structure du fichier smb.conf dans nano]

Exemple d'ajout d'un partage sécurisé à la fin du fichier :

```ini
[Partage_Doc]
   path = /var/shares/docs
   browseable = yes
   read only = no
   guest ok = no
   valid users = @techniciens
```

### Gestion des accès et utilisateurs

Samba utilise son propre système de mots de passe, distinct des mots de passe système Linux :

```bash
# Création du dossier et attribution des droits système
mkdir -p /var/shares/docs
chown -R root:techniciens /var/shares/docs
chmod -R 770 /var/shares/docs

# Ajout d'un utilisateur existant au SGBD Samba
smbpasswd -a loris

# Validation de la syntaxe et redémarrage du service
testparm
systemctl restart smbd nmbd
```

[image du résultat de la commande testparm validant la configuration]

[image du test d'accès au partage Samba depuis un client Windows]

---

## 2. NFS (Network File System)

!!! info "Présentation & Cas d'usage"
    **NFS** est un protocole de partage de fichiers natif aux environnements **Linux/Unix**. Il offre d'excellentes performances d'E/S et sert principalement à relier des serveurs de stockage (NAS), des hyperviseurs de virtualisation (Proxmox, VMware) ou des grappes de serveurs.

### Configuration du Serveur NFS

```bash
# Installation du serveur NFS
apt update && apt install -y nfs-kernel-server

# Création du répertoire à partager
mkdir -p /srv/nfs/data
chown -R nobody:nogroup /srv/nfs/data
chmod 777 /srv/nfs/data
```

Le paramétrage des accès s'effectue dans le fichier `/etc/exports` :

```text
/srv/nfs/data 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

Prise en compte de la configuration :

```bash
# Recharger la table d'exportation
exportfs -arv

# Redémarrer le service
systemctl restart nfs-kernel-server
```

[image du terminal exécutant la commande exportfs -arv]

### Configuration du Client NFS

```bash
# Installation du client sur une autre machine Linux
apt install -y nfs-common

# Afficher les partages disponibles sur le serveur
showmount -e 192.168.1.100

# Montage manuel du partage
mkdir -p /mnt/nfs_partage
mount -t nfs 192.168.1.100:/srv/nfs/data /mnt/nfs_partage
```

[image du résultat de la commande showmount affichant le partage disponible]

[image du dossier NFS monté et vérifié avec la commande df -h]

---

## 3. Clonezilla

!!! info "Présentation & Cas d'usage"
    **Clonezilla** est une solution open-source de déploiement et de sauvegarde d'images disque (*bare-metal*). Il permet de cloner un disque dur ou une partition complète, soit en local, soit à travers le réseau.

### Déclinaisons de Clonezilla

* **Clonezilla Live :** S'exécute depuis une clé USB ou un CD bootable. Idéal pour sauvegarder ou restaurer une seule machine à la fois.
* **Clonezilla SE (Server Edition) :** Intégré à un serveur PXE/DRBL pour déployer simultanément une même image système sur plusieurs dizaines de postes clients via le réseau (multicast).

[image du menu de démarrage de Clonezilla Live]

### Processus type de Sauvegarde / Restauration (Clonezilla Live)

1. **Démarrage :** Boot du poste cible sur le support Clonezilla Live (environnement Linux minimaliste).

2. **Choix du mode :**
   * `device-image` : Sauvegarde/Restauration d'un disque ou partition vers/depuis un fichier image.
   * `device-device` : Clonage direct de disque à disque (ex: migration vers un SSD).

[image de la sélection du mode device-image]

3. **Mise en place de l'emplacement d'image (`/home/partimag`) :**
   * L'image peut être stockée sur un disque USB externe ou sur le réseau via **NFS**, **Samba (SMB)** ou **SSH**.

[image du choix du serveur distant Samba ou NFS pour stocker l'image]

4. **Exécution des utilitaires sous-jacents :**
   * Clonezilla utilise `partclone` (ou `dd`, `ntfsclone`) pour ne copier que les blocs de données utilisés, ce qui optimise la taille des sauvegardes et le temps de transfert.

[image de l'écran de progression du clonage avec partclone]

---

## Synthèse et Comparatif des Technologies

| Outil | Domaine principal | OS Cibles | Points forts |
| :--- | :--- | :--- | :--- |
| **Samba** | Partage de fichiers & Gestion d'identité | Interplateforme (Windows / Linux / macOS) | Gestion des droits NTFS/ACL, intégration Active Directory. |
| **NFS** | Partage de fichiers haute performance | Linux / Unix | Faible empreinte CPU, idéal pour le stockage de VM et serveurs. |
| **Clonezilla** | Sauvegarde & Déploiement système | Agnostique (Ext4, NTFS, FAT, Btrfs...) | Copie bloc par bloc, ultra-rapide, déploiement réseau de masse. |




## Conexion via ssh


![TeamViewer prêt sur le PC1](../images/ssh1.png)