# Projet Jenkins - Fibonacci avec Agent Docker

Ce projet exécute un script shell calculant la suite de Fibonacci sur un agent Docker.

## Prérequis

✅ Jenkins Controller déjà en cours d'exécution (votre configuration existante)
✅ Docker Desktop installé et configuré
✅ Plugin Jenkins "Docker Pipeline" installé dans Jenkins

## Structure du Projet

```
03-jenkins-docker/
└── scripts/
    ├── fibonacci.sh    # Script de calcul de Fibonacci
    └── Jenkinsfile     # Pipeline Jenkins
```

## Intégration avec votre Jenkins existant

Votre Jenkins est déjà configuré avec le socket Docker monté, ce qui est parfait pour ce projet !

### Montage du projet dans votre Jenkins

Ajoutez ce volume à votre `docker-compose.yml` existant :

```yaml
volumes:
  - jenkins_home:/var/jenkins_home
  - /var/run/docker.sock:/var/run/docker.sock
  - //c/Formation/Learning-2026/Jenkins/03-jenkins-docker:/workspace/03-jenkins-docker  # ← Ajouter cette ligne
```

Puis redémarrez Jenkins :
```powershell
docker-compose restart jenkins
```

## Créer le Pipeline dans Jenkins

1. Accédez à votre Jenkins : **http://localhost:8080**
2. Cliquez sur **Nouvel Item** (ou **New Item**)
3. Entrez le nom : `Fibonacci-Docker-Agent`
4. Sélectionnez **Pipeline** et cliquez sur **OK**
5. Dans la configuration :

#### Option A : Pipeline depuis le workspace (Recommandé)
   - **Pipeline** → **Definition** : `Pipeline script`
   - Copiez-collez le contenu du fichier `scripts/Jenkinsfile`
   - Cliquez sur **Save**

#### Option B : Pipeline depuis SCM (si vous utilisez Git)
   - **Pipeline** → **Definition** : `Pipeline script from SCM`
   - **SCM** : Git
   - **Repository URL** : Votre URL Git
   - **Script Path** : `scripts/Jenkinsfile`

### 4. Configuration des Paramètres

Le pipeline accepte un paramètre personnalisable :
- **N** : Nombre de termes de la suite de Fibonacci (valeur par défaut : 10)

## Utilisation

### Lancer le pipeline

1. Dans Jenkins, ouvrez le job `Fibonacci-Docker-Agent`
2. Cliquez sur **Build with Parameters**
3. Entrez le nombre de termes souhaité (ex: 15)
4. Cliquez sur **Build**

### Ce qui se passe

1. Jenkins va télécharger l'image Docker `bash:latest` (si pas déjà présente)
2. Un container Docker temporaire sera créé comme agent
3. Le script `fibonacci.sh` sera exécuté dans ce container
4. Les résultats s'afficheront dans la console Jenkins
5. Le container sera automatiquement supprimé à la fin

### Exemple de sortie console

```
=== Informations de l'environnement ===
Workspace: /var/jenkins_home/workspace/Fibonacci-Docker-Agent
Nombre de termes Fibonacci: 10
✓ Script fibonacci.sh trouvé
=== Début du calcul de Fibonacci ===
The Fibonacci series is : 
0	0
1	1
2	1
3	2
4	3
5	5
6	8
7	13
8	21
9	34
=== Fin du calcul ===
✓ Pipeline exécuté avec succès!
```

## Résolution des Problèmes

### Le workspace n'est pas accessible dans le pipeline

**Solution** : Vérifiez que le volume est bien monté dans votre `docker-compose.yml` :
```yaml
- //c/Formation/Learning-2026/Jenkins/03-jenkins-docker:/workspace/03-jenkins-docker
```

### Erreur : "fibonacci.sh not found"

**Solution** : Assurez-vous que le fichier `fibonacci.sh` est bien dans le dossier `scripts/` et que Jenkins peut y accéder.

### Erreur : "Cannot connect to Docker daemon"

**Solution** : Votre configuration Jenkins monte déjà le socket Docker, mais vérifiez :
```powershell
docker exec jenkins ls -la /var/run/docker.sock
```

### L'image Docker bash:latest ne se télécharge pas

**Solution** : Vérifiez votre connexion internet et les logs :
```powershell
docker logs -f jenkins
```

## Commandes Utiles

```powershell
# Voir les logs Jenkins
docker logs -f jenkins

# Accéder au container Jenkins
docker exec -it jenkins bash

# Lister les images Docker disponibles
docker images

# Voir les containers en cours
docker ps

# Redémarrer Jenkins
docker-compose restart jenkins
```

## Architecture du Projet

```
┌─────────────────────────────────────┐
│   Docker Desktop (Windows)          │
│                                     │
│  ┌──────────────────────────────┐  │
│  │  Jenkins Controller          │  │
│  │  (Container: jenkins)        │  │
│  │                              │  │
│  │  ┌────────────────────────┐  │  │
│  │  │  Pipeline Job          │  │  │
│  │  │  Fibonacci-Docker      │  │  │
│  │  └────────────────────────┘  │  │
│  └──────────┬───────────────────┘  │
│             │                       │
│             │ Crée un agent Docker  │
│             ↓                       │
│  ┌──────────────────────────────┐  │
│  │  Agent Docker                │  │
│  │  (Container temporaire)      │  │
│  │  Image: bash:latest          │  │
│  │                              │  │
│  │  Exécute: fibonacci.sh       │  │
│  └──────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

## Avantages de cette Architecture

✅ **Isolation** : Chaque build s'exécute dans un container isolé
✅ **Reproductibilité** : Environnement identique à chaque exécution
✅ **Nettoyage automatique** : Les containers agents sont supprimés après usage
✅ **Scalabilité** : Possibilité d'exécuter plusieurs builds en parallèle
✅ **Pas de pollution** : L'environnement Jenkins reste propre
