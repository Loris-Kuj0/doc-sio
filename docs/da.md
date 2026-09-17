# Documentation : Accès à distance

**Auteur :** Loris.R  
**Module :** B2  
**Sujet :** Mettre en œuvre trois méthodes d'accès à distance sous Linux

---

!!! abstract "Notes"
    Ce guide présente l'installation, la configuration et le cas d'usage pratique de trois méthodes d'accès à distance aux philosophies distinctes : une solution propriétaire commerciale (**TeamViewer / QuickSupport**), une alternative open source légère (**RustDesk** au format AppImage) et un tandem haute performance initialement dédié au streaming et au cloud gaming (**Sunshine / Moonlight**).

---

## Topologie réseau

Pour l'ensemble des démonstrations présentées dans cette documentation, l'infrastructure s'articule autour de trois machines réparties sur deux sous-réseaux distincts :

* **Sous-réseau local 1 (`192.168.1.0/24`) :**
    * **PC1 (`192.168.1.91`) :** Fedora GNOME / Bazzite — agit comme la station de contrôle centrale (client).
    * **PC2 (`192.168.1.31`) :** Fedora KDE / Bazzite — poste cible situé dans le même segment réseau que le client.
* **Sous-réseau virtuel Proxmox (`192.168.209.0/24`) :**
    * **PC3 (`192.168.209.151`) :** Machine virtuelle sous Debian 13 (KDE Plasma) hébergée sur Proxmox. Il s'agit d'un poste cible distant routé hors du réseau local de départ.

```mermaid
graph TD
    subgraph LAN1 ["Réseau Local (192.168.1.0/24)"]
        R1[Routeur Local<br/>192.168.1.254]
        PC1[PC1 - Fedora GNOME / Bazzite<br/>192.168.1.91<br/><i>Station Client</i>]
        PC2[PC2 - Fedora KDE / Bazzite<br/>192.168.1.31<br/><i>Poste Cible Local</i>]
        
        R1 <--> PC1
        R1 <--> PC2
    end

    subgraph PROXMOX ["Infra Proxmox (192.168.209.0/24)"]
        GW[Passerelle Virtuelle]
        PC3[PC3 - VM Debian 13 KDE<br/>192.168.209.151<br/><i>Poste Cible Distant</i>]
        
        GW <--> PC3
    end

    PC1 -.->|Session locale| PC2
    PC1 -.->|Session distante| PC3
```

---

## 1. TeamViewer & QuickSupport

!!! note "Périmètre d'accès"
    * **Machine cliente (contrôle) :** PC1 (`192.168.1.91` — Bazzite)
    * **Machine cible (administrée) :** PC2 (`192.168.1.31` — Bazzite KDE)

!!! info "Différence entre TeamViewer complet et QuickSupport"
    * **TeamViewer (Client complet) :** logiciel lourd incluant le démon système (`teamviewerd`), la gestion de comptes centralisée, la prise en main entrante et sortante, ainsi que la gestion de carnet d'adresses.
    * **QuickSupport (Module d'assistance) :** application légère, portable et sans installation obligatoire. Elle est conçue exclusivement pour recevoir une assistance ponctuelle : elle génère un ID unique et un mot de passe temporaire dès son lancement sans altérer le système.

---

### A. Déploiement sur le poste cible (PC2)

Sur le **PC2 (Fedora KDE)**, la version légère **QuickSupport** a été récupérée sous forme d'archive compressée (`tar.gz`) directement depuis le site officiel de TeamViewer, puis extraite pour une exécution à la demande.

```bash
# Extraction de l'archive tarball
tar -xvf teamviewer_quicksupport.tar.gz

# Lancement de l'exécutable portable
./teamviewerqs/quicksupport
```

![Lancement de QuickSupport sur le PC2](../images/da1.png)

---

### B. Installation avancée sur la station client (PC1)

!!! warning "Contrainte d'architecture immuable (Bazzite / rpm-ostree)"
    Le **PC1** fonctionne sous **Bazzite** (système à image immuable basé sur Fedora Silverblue). Sur ce type de distribution, le répertoire système `/usr` est monté en lecture seule. On ne peut pas utiliser un gestionnaire classique comme `dnf`. L'installation de paquets natifs nécessite de superposer (*layering*) le paquet RPM dans l'image système via `rpm-ostree`, suivi d'un redémarrage.

#### Étape 1 : Téléchargement du paquet RPM officiel
Le paquet RPM 64 bits est récupéré dans l'espace temporaire du système :

```bash
curl -fL -o /tmp/teamviewer.rpm "[https://download.teamviewer.com/download/linux/teamviewer.x86_64.rpm](https://download.teamviewer.com/download/linux/teamviewer.x86_64.rpm)"
```

*Vérification :* s'assurer que le fichier téléchargé pèse environ 60 à 80 Mo via la commande `ls -lh /tmp/teamviewer.rpm`.

#### Étape 2 : Superposition du paquet via rpm-ostree
Le paquet est directement injecté dans l'arborescence immuable :

```bash
rpm-ostree install /tmp/teamviewer.rpm
```

*Effet :* `rpm-ostree` prépare une nouvelle image système fusionnée (*Changes queued for next boot*) qui sera chargée au prochain démarrage.

#### Étape 3 : Application du nouveau déploiement système
Redémarrage de la machine pour basculer sur la nouvelle image hôte contenant TeamViewer :

```bash
systemctl reboot
```

*Vérification :* après le boot, la commande `rpm-ostree status` doit afficher `teamviewer` dans la section `LayeredPackages`.

#### Étape 4 : Déclaration et activation du service systemd
Le démon de TeamViewer (`teamviewerd`) doit être exécuté en tâche de fond. Étant stocké dans `/opt/teamviewer/tv_bin/script/`, il convient de copier son unité dans le répertoire modifiable `/etc/systemd/system/` :

```bash
# 1. Copie du fichier de service systemd
sudo cp /opt/teamviewer/tv_bin/script/teamviewerd.service /etc/systemd/system/

# 2. Rechargement du gestionnaire de services
sudo systemctl daemon-reload

# 3. Activation et démarrage immédiat du démon
sudo systemctl enable --now teamviewerd.service
```

*Vérification :* la commande `systemctl status teamviewerd.service` doit retourner l'état `Active: active (running)`.

#### Étape 5 : Lancement et connexion

L'application est exécutée sur le **PC1** (via le terminal avec la commande `teamviewer` ou depuis le menu des applications) :

![TeamViewer prêt sur le PC1](../images/da2.png)

Une fois l'interface ouverte et le compte utilisateur connecté, il suffit de renseigner l'**ID partenaire** et le **mot de passe** fournis à l'écran par QuickSupport sur le PC2 pour initier le contrôle à distance.

---

### C. Démonstration vidéo : TeamViewer

<video width="100%" controls>
  <source src="../images/v1.mp4" type="video/mp4">
</video>

---

### D. Procédure de désinstallation / nettoyage (maintenance)

Pour retirer proprement la couche logicielle sur un système immuable :

```bash
# Arrêt et suppression du service démon
sudo systemctl disable --now teamviewerd.service
sudo rm -f /etc/systemd/system/teamviewerd.service
sudo systemctl daemon-reload

# Retrait de la couche logicielle de l'image immuable
rpm-ostree uninstall teamviewer

# Redémarrage pour appliquer le nettoyage
systemctl reboot
```

---

## 2. RustDesk (Solution open source portable)

!!! note "Périmètre d'accès"
    * **Machine cliente (contrôle) :** PC1 (`192.168.1.91` — Bazzite)
    * **Machine cible (administrée) :** PC3 (`192.168.209.151` — VM Debian 13 KDE sur Proxmox)

### Mettre en œuvre RustDesk

Contrairement à la lourdeur de déploiement de TeamViewer, RustDesk est disponible sous forme d'**AppImage** (un format d'application que j'affectionne tout particulièrement) sur les deux postes en quelques clics :

1. Téléchargement du binaire `.AppImage` depuis le dépôt officiel GitHub de RustDesk.
2. Octroi des droits d'exécution : `chmod +x rustdesk-*.AppImage`.
3. Lancement direct par double-clic ou via le terminal.

!!! tip "L'intérêt du format AppImage sur un OS immuable ou une VM"
    L'**AppImage** est un format d'encapsulation universel pour Linux. Il regroupe l'application et l'ensemble de ses dépendances dans un seul fichier binaire exécutable. Il ne nécessite **aucune installation**, aucun privilège d'administrateur (`root`), et n'altère en rien l'image système. C'est le format idéal aussi bien pour une distribution atomique comme Bazzite que pour un déploiement rapide sur une VM Debian 13.

#### Interface sur le client (PC1) :
![Interface RustDesk sur PC1](../images/da3.png)

#### Interface sur la cible (PC3) :
![Interface RustDesk sur PC3](../images/da5.png)

Il suffit ensuite de saisir l'identifiant du PC3 dans le champ du PC1, puis de renseigner le mot de passe de session généré par RustDesk pour établir la connexion à travers les réseaux routés.

---

### Démonstration vidéo : RustDesk

<video width="100%" controls>
  <source src="../images/v2.mp4" type="video/mp4">
</video>

---

## 3. Sunshine & Moonlight (Accès haute performance / faible latence)

!!! note "Périmètre d'accès"
    * **Machine cliente (contrôle) :** PC1 (`192.168.1.91` — Bazzite)
    * **Machine cible (administrée) :** PC2 (`192.168.1.31` — Bazzite KDE)

!!! info "Présentation et détournement de cas d'usage"
    * **Sunshine :** serveur hôte open source de flux vidéo/audio auto-hébergé (remplaçant l'ancien NVIDIA GameStream). Il s'installe sur la machine distante à contrôler (PC2).
    * **Moonlight :** client de réception ultra-léger et optimisé pour le décodage matériel. Il s'installe sur la machine de contrôle (PC1).

### Pourquoi utiliser cette solution en administration système ?

Bien que le couple Sunshine / Moonlight soit initialement conçu pour le streaming de jeux vidéo en réseau local, son architecture offre des avantages inédits pour l'administration à distance :

* **Latence quasi nulle :** enregistrement et encodage vidéo matériel direct (via GPU NVENC, VAAPI ou QuickSync) permettant un flux à 60 ou 120 FPS constant.
* **Fluidité d'affichage exceptionnelle :** supérieure aux protocoles classiques comme le VNC ou le RDP lors du traitement de tâches graphiques lourdes.
* **Sécurité :** appairage chiffré via un code PIN à quatre chiffres lors de la première poignée de main entre le client Moonlight et le serveur Sunshine.

Une fois l'appairage effectué, Moonlight permet d'afficher l'intégralité du bureau distant du PC2 avec une réactivité identique à celle d'un moniteur branché physiquement sur la machine.

---

## 4. Tableau comparatif des solutions

Pour synthétiser les spécificités de chaque outil et guider le choix de la solution selon le besoin d'administration, voici un comparatif de leurs caractéristiques:

| Critère | TeamViewer & QuickSupport | RustDesk | Sunshine & Moonlight |
| :--- | :--- | :--- | :--- |
| **Licence & Philosophie** | Propriétaire / Commercial (Gratuit usage privé) | Open source (AGPLv3) & Auto-hébergeable | Open source (GPLv3) & 100 % Auto-hébergé |
| **Environnement Réseau** | **LAN & WAN** (Traversée transparente des pare-feu via serveurs relais cloud propriétaires) | **LAN & WAN** (Serveurs de signalement publics ou serveur privé auto-hébergé) | **LAN prioritaire** (Accès WAN possible via VPN/Tailscale ou redirection de ports) |
| **Type de Déploiement** | **Client lourd :** installation système (RPM/démon)<br>**QuickSupport :** binaire portable | **AppImage autonome** (Exécution directe sans installation ni privilèges) | **Serveur :** démon système de capture hôte<br>**Client :** application réceptrice légère |
| **Performance & Latence** | Standard (Optimisé pour la bureautique et le transfert de fichiers) | Bonne à Très bonne (Ajustable selon le relais utilisé et le codec choisi) | **Ultra-haute performance** (Encodage matériel GPU, 60/120 FPS, latence imperceptible) |
| **Prérequis Cible** | Interaction utilisateur requise (Transmission ID / Mot de passe temporaire) | Configuration flexible (Accès permanent ou temporaire via mot de passe) | Configuration préalable (Service actif et appairage PIN initial requis) |
| **Cas d'usage idéal** | **Support utilisateur à chaud** et assistance ponctuelle grand public | **Support IT régulier**, administration système et machines immuables | **Station de travail graphique**, CAO/3D, montage vidéo à distance, Cloud Gaming |