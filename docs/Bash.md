# Documentation : Scripting en Bash

---

# 0.

Ce guide présente les bases fondamentales pour créer des scripts d'automatisation sous Linux.

## 1. Structure d'un script
Tout script Bash doit commencer par le **Shebang**. Cette ligne indique au système que le fichier doit être interprété par Bash.

```bash
#!/bin/bash

# Ceci est un commentaire
echo "Vous êtes en BTS SIO"
```

### Exécution
Pour qu'un script puisse être lancé, il doit posséder les droits d'exécution :

```bash
# Donner les droits
chmod +x mon_script.sh

# Lancer le script
./mon_script.sh
```

!!! tip "Astuce"
    Utilisez l'extension `.sh` pour identifier vos scripts, bien que ce ne soit pas obligatoire sous Linux.

## 2. Variables et Entrées
En Bash, la gestion des espaces est cruciale : **aucun espace** autour du signe `=` lors de l'assignation.

```bash
#!/bin/bash

# Déclaration
NOM="Loris"
echo "Vous êtes en BTS SIO, $NOM"

# Lecture d'une entrée utilisateur
echo "Quel âge as-tu ?"
read AGE
echo "Tu as ${AGE}ans."
```
![BASH1](../images/Bash1.png)

## 3. Les Conditions
Les conditions s'écrivent entre crochets `[ ]`. **Attention :** les espaces après `[` et avant `]` sont obligatoires.

### Comparateurs numériques
| Opérateur | Signification |
| :--- | :--- |
| `-eq` | Égal à (Equal) |
| `-ne` | Différent de (Not equal) |
| `-gt` | Plus grand que (Greater than) |
| `-lt` | Plus petit que (Less than) |
| `-ge` | Supérieur ou égal |
| `-le` | Inférieur ou égal |

### Exemple
```bash
if [ $AGE -ge 18 ]; then
    echo "Accès autorisé : vous êtes majeur."
else
    echo "Accès refusé : vous êtes mineur."
fi
```

## 4. Les Boucles
Les boucles permettent de répéter des tâches sur des listes de fichiers ou des plages de nombres.

### Boucle For
```bash
# Boucle sur une plage de nombres
for i in {1..5}; do
    echo "Étape $i"
done

# Boucle sur les fichiers du répertoire courant
for fichier in *.txt; do
    echo "Traitement de $fichier"
done
```

### Boucle While
```bash
COMPTEUR=1
while [ $COMPTEUR -le 3 ]; do
    echo "Compte : $COMPTEUR"
    ((COMPTEUR++))
done
```

## 5. Gestion des Arguments
Vous pouvez passer des paramètres à votre script lors de son appel : `./script.sh arg1 arg2`.

| Variable | Description |
| :--- | :--- |
| `$0` | Nom du script |
| `$1` à `$9` | Premier, deuxième argument, etc. |
| `$#` | Nombre total d'arguments passés |
| `$@` | Liste complète des arguments |
| `$?` | Code de retour de la dernière commande (0 = succès) |

## 6. Exemple complet : Backup automatique
Voici un script récapitulatif pour sauvegarder un dossier.

```bash
#!/bin/bash

SOURCE="/home/user/documents"
DESTINATION="/home/user/backups"
DATE=$(date +%Y-%m-%d)

# Vérification de l'existence du dossier source
if [ -d "$SOURCE" ]; then
    echo "--- Début de la sauvegarde ---"
    tar -czf "$DESTINATION/backup_$DATE.tar.gz" "$SOURCE"
    
    if [ $? -eq 0 ]; then
        echo "Succès : Fichier sauvegardé dans $DESTINATION"
    else
        echo "Erreur : La compression a échoué"
    fi
else
    echo "Erreur : Le dossier source n'existe pas."
    exit 1
fi
```

!!! warning "Attention"
    Testez toujours vos scripts dans un environnement sécurisé avant de les utiliser sur des données de production.