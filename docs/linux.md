# Documentation : Gestion des Sauvegardes et de la Continuite d'Activite

!!! abstract "Introduction : Objectifs"
    La sauvegarde informatique est le pilier central de la securite des donnees.

## Obligations Legales et Durees de Conservation

La conservation des donnees ne releve pas uniquement d'un choix technique, mais d'un cadre reglementaire strict impose par l'Etat francais (RGPD, Code de commerce, Code du travail). Les organisations ont l'obligation de conserver certains documents pour des durées minimales.

* **Documents civils et commerciaux :** Les contrats conclus dans le cadre d'une relation commerciale ou les factures clients/fournisseurs doivent etre conserves pendant **10 ans**.
* **Donnees RH et administratives :** Les bulletins de paie ou les contrats de travail doivent etre conserves pendant **5 ans**.
* **Donnees de sante ou bancaires :** Soumises a des reglementations encore plus strictes, exigeant parfois des durees allant jusqu'a 20 ans ou plus, gerees par des hebergeurs certifies.

!!! warning "Risque Juridique"
    Le non-respect de ces durees de conservation ou l'incapacite a restituer ces donnees lors d'un controle ou d'un litige expose l'organisation a de lourdes sanctions financieres et penales.

---

## Distinction Fondamentale : Sauvegarde vs Archivage

Une erreur frequente en administration systeme consiste a confondre la sauvegarde et l'archivage. Bien que ces deux processus manipulent les memes donnees, leurs finalites sont differentes.

| Caracteristique | Sauvegarde (Backup) | Archivage |
| :--- | :--- | :--- |
| **Objectif Principal** | Restauration rapide apres un incident (panne, cyberattaque, erreur humaine). | Conservation a long terme pour des raisons legales ou historiques. |
| **Etat de la Donnee** | Donnees vivantes, dynamiques, qui changent frequemment. | Donnees figees, immuables, qui ne seront plus modifiees. |
| **Accessibilite** | Acces immediat et transparent pour les administrateurs/utilisateurs. | Acces ponctuel, souvent indexe pour des recherches specifiques. |
| **Cycle de vie** | Retention courte a moyenne (les anciennes versions ecrasent les nouvelles). | Retention definitive ou a tres long terme (plusieurs annees/decennies). |

!!! tip "Pour faire tres simple."
    * **Sauvegarde :** "J'ai perdu mon fichier ce matin, je dois le recuperer dans son etat d'hier."
    * **Archivage :** "Le fisc me demande les factures de l'annee 2018, je dois les extraire du coffre-fort numerique."

---

## Typologies de Sauvegardes

Le choix d'une methode de sauvegarde depend de l'equilibre a trouver entre la fenetre de sauvegarde disponible (le temps alloue pour copier les donnees) et le volume de stockage disponible sur les serveurs cibles.

### La Sauvegarde Complete
Ce mecanisme consiste a copier l'integralite des donnees selectionnees a chaque cycle, independamment de toute modification anterieure.

* **Avantages :** * Restauration ultra-rapide et simplifiee (un seul jeu de donnees est necessaire).
    * Independance totale de la sauvegarde par rapport aux cycles precedents.
* **Inconvenients :**
    * Consommation d'espace de stockage critique et redondante.
    * Temps d'execution tres long (fenetre de sauvegarde elevee).
* *Effet :* Ideale comme point de depart (generalement planifiee une fois par semaine, le week-end).

### La Sauvegarde Incrementale
Ce processus ne sauvegarde que les donnees creees ou modifiees depuis la **derniere sauvegarde**, qu'elle soit complete ou incrementale.

* **Avantages :**
    * Execution extremement rapide.
    * Espace de stockage requis minimal.
* **Inconvenients :**
    * Restauration lente et complexe. En cas de sinistre, il est necessaire de restaurer la derniere sauvegarde complete, puis **chaque increment successif** dans l'ordre chronologique sans aucune rupture de la chaine.
* *Effet :* Reduit drastiquement la charge reseau au quotidien (planifiee chaque nuit).

### La Sauvegarde Differentielle
Ce mecanisme copie uniquement les donnees modifiees depuis la **derniere sauvegarde complete**.

* **Avantages :**
    * Restauration plus rapide que l'incrementale (necessite uniquement la sauvegarde complete initiale + la derniere sauvegarde differentielle en date).
    * Espace requis modere (inferieur a une complete).
* **Inconvenients :**
    * La taille de la sauvegarde augmente de jour en jour a mesure qu'on s'eloigne de la derniere sauvegarde complete, allongeant progressivement le temps d'execution quotidien.
* *Effet :* Represente un excellent compromis entre rapidite de restauration et optimisation de l'espace de stockage.

---

## TP : Programmation d'une Sauvegarde dans le Cloud

### A. Specifications de l'Environnement de Travail
La realisation technique de ce cas pratique s'effectue sur un poste client disposant de l'environnement materiel et logiciel suivant :

* **Systeme d'exploitation :** Bazzite OS (generation 43 / Base Fedora Silverblue)
* **Interface graphique :** KDE Plasma
* **Contrainte d'architecture :** Systeme de fichiers immuable (systeme racine monte en lecture seule).

### B. Logiciel Choisi
Le choix s'est porte sur **Rclone**, un utilitaire open-source en ligne de commande hautement performant specialise dans la gestion, la synchronisation et le transfert de donnees vers des infrastructures de stockage Cloud (IaaS). En raison de l'immuabilite de l'OS hote, le binaire a ete deploye manuellement de maniere isolee dans le repertoire personnel de l'utilisateur (`~/.local/bin`) afin d'assurer sa persistance.

---

### C. Procedure Technique et Implementation

#### 1. Deploiement et initialisation du binaire Rclone
N'ayant pas la possibilite d'utiliser un gestionnaire de paquets global sans alterer l'image du systeme, le binaire autonome est recupere directement depuis les depots officiels du projet :

```bash
# Telechargement et decompression du package dans l'espace temporaire
cd /tmp && wget [https://downloads.rclone.org/rclone-current-linux-amd64.zip](https://downloads.rclone.org/rclone-current-linux-amd64.zip) && unzip rclone-current-linux-amd64.zip

# Isolation du fichier executable dans l'environnement de l'utilisateur lk
mkdir -p ~/.local/bin && cp /tmp/rclone-*-linux-amd64/rclone ~/.local/bin/ && chmod +x ~/.local/bin/rclone

# Enregistrement temporaire du dossier cible dans la variable PATH du terminal
export PATH="$HOME/.local/bin:$PATH"

```

```bash
# Verification de l'integrite operationnelle du programme
rclone version

```

*Effet :* La version `v1.74.1` repond correctement aux appels du terminal.

![Validation de la version locale rclone](../images/Screenshot_2026-05-20_18-56-47.jpg)

#### 2. Configuration du point d'acces distant (Remote Google Drive)

L'interfacage avec l'espace Cloud s'effectue via l'assistant de configuration interactif :

```bash
rclone config

```

* **Creation du profil :** Selection de l'option `n` (*New remote*) nommee logiquement `tpsio`.
* **Choix du stockage :** Selection de l'option correspondant a Google Drive.
* **Gestion des permissions (`scope`) :** Attribution du profil `1` (*Full access all files*) pour autoriser l'edition complete sur le Drive cible.
* **Authentification unifiee (OAuth2) :** Utilisation de l'authentification automatique (`Use auto config? -> y`).

Des invites de commande specifiques se presentent durant l'initialisation :

```text
Enter value. Press Enter to leave empty.
client_secret> 

Option scope.
Choose a number from below, or type in your own value.
 1 / Full access all files, excluding Application Data Folder.
   \ (drive)
scope> 1

```

Le programme initie un serveur d'ecoute local et bascule la validation sur le navigateur web de la session pour approuver l'acces au compte Google.

L'accord d'acces est alors confirme par le serveur d'authentification :

Une fois l'autorisation validee par l'API Google, le terminal recoit le jeton de securite (`Got code`), refuse la creation d'un disque d'equipe partage (`Shared Drive -> n`), affiche le recapitulatif chiffre du profil et ferme l'utilitaire via la commande `q` (*Quit*).

L'enregistrement final montre la bonne creation du profil `tpsio` :

#### 3. Ecriture du Script d'Automatisation Bash

Un script d'ordonnancement est redige a l'aide de l'editeur de texte **Kate** pour automatiser le traitement iteratif des repertoires du module.

!!! warning "Sensibilite a la Casse (Case Sensitivity)"
Le chemin absolu d'acces de l'infrastructure de dossiers de TP doit imperativement respecter la casse reelle de l'architecture Linux, a savoir : `/home/LK/SIO-TP`. Toute divergence typographique (ex: minuscules) provoque la levee d'une exception d'inexistence des sources.

```bash
#!/bin/bash

# Configuration des chemins locaux (Casse stricte : LK) et distants
SOURCE_DIR="/home/LK/SIO-TP"
REMOTE_TARGET="tpsio:Sauvegardes_SIO"

# Liste sequentielle des repertoires cibles du module
REPERTOIRES=("AP" "B1" "B2" "B3")

echo "===================================================="
echo " DEBUT DE LA SAUVEGARDE SYNCHRONE VERS GOOGLE DRIVE"
echo "===================================================="

for dossier in "${REPERTOIRES[@]}"
do
    if [ -d "$SOURCE_DIR/$dossier" ]; then
        echo "--> Traitement du repertoire : $dossier"
        
        # Duplication selective des nouveaux assets et des fichiers modifies
        ~/.local/bin/rclone copy "$SOURCE_DIR/$dossier" "$REMOTE_TARGET/$dossier" --progress
    else
        echo "[ERREUR] Le repertoire local $SOURCE_DIR/$dossier n'existe pas."
    fi
done

echo "===================================================="
echo "        SAUVEGARDE TERMINÉE AVEC SUCCÈS !"
echo "===================================================="

```

L'edition s'effectue dans l'environnement de developpement :

#### 4. Attribution des droits et execution de la routine

Avant de pouvoir executer le script, il est imperatif de modifier ses permissions systeme pour lui octroyer le privilege d'execution :

```bash
chmod +x backup.sh

```

Le lancement du script execute la routine en arriere-plan. Rclone analyse l'arborescence locale, identifie les fichiers et televerse de maniere ciblee le contenu vers l'espace IaaS.

```bash
./backup.sh

```

Le script confirme le bon traitement de chaque sous-dossier cible :

#### 5. Controle de conformite de l'infrastructure Cloud

La verification s'opere en accedant a l'interface d'administration Web de Google Drive. Les repertoires distants ont ete crees et le fichier de verification `test.txt` est correctement synchronise dans l'arborescence cible.

Une vue detaillee confirme la presence et l'heure exacte de synchronisation de l'element :

```

***

### 2. Liste des balises d'images (à copier séparément)

Voici les lignes de code brutes de tes images. Tu peux copier chaque ligne et remplacer la balise correspondante `` directement dans ton fichier Markdown :

* **Pour `` :**
```markdown


```

* **Pour `` :**

```markdown
![Parametres du type de stockage cloud](../images/Screenshot_2026-05-20_19-04-31.jpg)

```

* **Pour `` :**

```markdown
![Ouverture de la mire de selection Google](../images/Screenshot_2026-05-20_19-04-50.jpg)

```

* **Pour `` :**

```markdown
![Succes de l authentification web](../images/Screenshot_2026-05-20_19-05-35.png)

```

* **Pour `` :**

```markdown
![Recuperation du code oAuth](../images/Screenshot_2026-05-20_19-06-42.jpg)

```

* **Pour `` :**

```markdown
![Validation finale du remote tpsio](../images/Screenshot_2026-05-20_19-08-25.jpg)

```

* **Pour `` :**

```markdown
![Edition du script Bash dans Kate](../images/Screenshot_2026-05-20_20-24-05.jpg)

```

* **Pour `` :**

```markdown
![Application du chmod sur le terminal](../images/Screenshot_2026-05-20_20-26-20.jpg)

```

* **Pour `` :**

```markdown
![Execution reussie du transfert synchrone](../images/Screenshot_2026-05-20_20-36-32.jpg)

```

* **Pour `` :**

```markdown
![Verification du fichier envoye sur Google Drive](../images/Screenshot_2026-05-20_20-36-50.jpg)

```

* **Pour `` :**

```markdown
![Metadonnees de synchronisation du fichier test](../images/Screenshot_2026-05-20_20-36-50.tmp.png)

```