# Documentation: Gestion des Sauvegardes et de la Continuité d'Activité

!!! abstract "Introduction : Objectifs"
    La sauvegarde informatique est le pilier central de la sécurité des données.

## 1. Obligations Légales et Durées de Conservation

La conservation des données ne relève pas uniquement d'un choix technique, mais d'un cadre réglementaire strict imposé par l'État français (RGPD, Code de commerce, Code du travail). Les organisations ont l'obligation de conserver certains documents pour des durées minimales.

* **Documents civils et commerciaux :** Les contrats conclus dans le cadre d'une relation commerciale ou les factures clients/fournisseurs doivent être conservés pendant **10 ans** (Code de commerce).
* **Données RH et administratives :** Les bulletins de paie ou les contrats de travail doivent être conservés pendant **5 ans**.
* **Données de santé ou bancaires :** Soumises à des réglementations encore plus strictes, exigeant parfois des durées allant jusqu'à 20 ans ou plus, gérées par des hébergeurs certifiés.

!!! warning "Risque Juridique"
    Le non-respect de ces durées de conservation ou l'incapacité à restituer ces données lors d'un contrôle ou d'un litige expose l'organisation à de Lourdes sanctions financières et pénales.
    

---

## 2. Distinction Fondamentale : Sauvegarde vs Archivage

Une erreur fréquente en administration système consiste à confondre la sauvegarde et l'archivage. Bien que ces deux processus manipulent les mêmes données, leurs finalités et leurs mécanismes techniques sont radicalement différents.

| Caractéristique | Sauvegarde (Backup) | Archivage |
| :--- | :--- | :--- |
| **Objectif Principal** | Restauration rapide après un incident (panne, cyberattaque, erreur humaine). | Conservation à long terme pour des raisons légales ou historiques. |
| **État de la Donnée** | Données vivantes, dynamiques, qui changent fréquemment. | Données figées, immuables, qui ne seront plus modifiées. |
| **Accessibilité** | Accès immédiat et transparent pour les administrateurs/utilisateurs. | Accès ponctuel, souvent indexé pour des recherches spécifiques. |
| **Cycle de vie** | Rétention courte à moyenne (les anciennes versions écrasent les nouvelles). | Rétention définitive ou à très long terme (plusieurs années/décennies). |

!!! tip "Pour faire tres simple."
    * **Sauvegarde :** "J'ai perdu mon fichier ce matin, je dois le récupérer dans son état d'hier."
    * **Archivage :** "Le fisc me demande les factures de l'année 2018, je dois les extraire du coffre-fort numérique."

---

## 3. Typologies de Sauvegardes : Mécanismes, Avantages et Inconvénients

Le choix d'une méthode de sauvegarde dépend de l'équilibre à trouver entre la fenêtre de sauvegarde disponible (le temps alloué pour copier les données) et le volume de stockage disponible sur les serveurs cibles.

### A. La Sauvegarde Complète
Ce mécanisme consiste à copier l'intégralité des données sélectionnées à chaque cycle, indépendamment de toute modification antérieure.

* **Avantages :** * Restauration ultra-rapide et simplifiée (un seul jeu de données est nécessaire).
    * Indépendance totale de la sauvegarde par rapport aux cycles précédents.
* **Inconvénients :**
    * Consommation d'espace de stockage critique et redondante.
    * Temps d'exécution très long (fenêtre de sauvegarde élevée).
* *Effet :* Idéale comme point de départ (généralement planifiée une fois par semaine, le week-end).

### B. La Sauvegarde Incrémentale
Ce processus ne sauvegarde que les données créées ou modifiées depuis la **dernière sauvegarde**, qu'elle soit complète ou incrémentale.

* **Avantages :**
    * Exécution extrêmement rapide.
    * Espace de stockage requis minimal.
* **Inconvénients :**
    * Restauration lente et complexe. En cas de sinistre, il est nécessaire de restaurer la dernière sauvegarde complète, puis **chaque incrément successif** dans l'ordre chronologique sans aucune rupture de la chaîne.
* *Effet :* Réduit drastiquement la charge réseau au quotidien (planifiée chaque nuit).

### C. La Sauvegarde Différentielle
Ce mécanisme copie uniquement les données modifiées depuis la **dernière sauvegarde complète**.

* **Avantages :**
    * Restauration plus rapide que l'incrémentale (nécessite uniquement la sauvegarde complète initiale + la dernière sauvegarde différentielle en date).
    * Espace requis modéré (inférieur à une complète).
* **Inconvénients :**
    * La taille de la sauvegarde augmente de jour en jour à mesure qu'on s'éloigne de la dernière sauvegarde complète, allongeant progressivement le temps d'exécution quotidien.
* *Effet :* Représente un excellent compromis entre rapidité de restauration et optimisation de l'espace de stockage.

---

## 4. TP : Programmation d'une Sauvegarde dans le Cloud

-Environement (linux/spec du pc/distro)
-Logiciel(s) choisi

-procedure

-a la toute fin explique moi comment mettre une video 

