# Configuration Ngrok pour Jenkins

## Installation de Ngrok

### Option 1 : Via Chocolatey (Recommandé pour Windows)
```powershell
choco install ngrok
```

### Option 2 : Téléchargement Manuel
1. Visitez https://ngrok.com/download
2. Téléchargez la version Windows
3. Extrayez `ngrok.exe` dans un dossier (ex: `C:\ngrok\`)
4. Ajoutez le dossier au PATH système

## Configuration

### 1. Créer un compte Ngrok (Gratuit)
- Allez sur https://dashboard.ngrok.com/signup
- Créez votre compte
- Récupérez votre **authtoken**

### 2. Configurer l'authtoken
```powershell
ngrok config add-authtoken VOTRE_TOKEN_ICI
```

## Exposer Jenkins avec Ngrok

### Méthode Simple (Port 8080)
```powershell
ngrok http 8080
```

### Avec un sous-domaine personnalisé (Plan Payant)
```powershell
ngrok http --domain=votre-domaine.ngrok.io 8080
```

### Avec authentification basique
```powershell
ngrok http 8080 --basic-auth="user:password"
```

## Script PowerShell pour démarrer Ngrok

Créez un fichier `start-ngrok.ps1` :

```powershell
# Démarrer ngrok pour Jenkins
Write-Host "🚀 Démarrage de ngrok pour Jenkins..." -ForegroundColor Green

# Vérifier si Jenkins est en cours d'exécution
$jenkinsRunning = docker ps --filter "name=jenkins" --format "{{.Names}}" 2>$null
if (-not $jenkinsRunning) {
    Write-Host "⚠️  Jenkins n'est pas en cours d'exécution!" -ForegroundColor Yellow
    Write-Host "Démarrez Jenkins d'abord avec: docker-compose up -d" -ForegroundColor Yellow
    exit 1
}

Write-Host "✓ Jenkins détecté" -ForegroundColor Green
Write-Host "`nExposition de Jenkins sur Internet via ngrok..." -ForegroundColor Cyan
Write-Host "Port exposé: 8080" -ForegroundColor White

# Démarrer ngrok
ngrok http 8080
```

## Utilisation

### 1. Démarrer Jenkins
```powershell
docker-compose up -d
```

### 2. Démarrer Ngrok
```powershell
ngrok http 8080
```

### 3. Récupérer l'URL publique
Ngrok affichera quelque chose comme :
```
Forwarding    https://abc123.ngrok.io -> http://localhost:8080
```

Votre Jenkins sera accessible via : `https://abc123.ngrok.io`

## Configuration Jenkins avec Ngrok

### Mettre à jour l'URL Jenkins
1. Connectez-vous à Jenkins
2. Allez dans **Manage Jenkins** → **Configure System**
3. Trouvez **Jenkins URL**
4. Remplacez par votre URL ngrok : `https://abc123.ngrok.io/`
5. Sauvegardez

## Ngrok Dashboard Web

Pendant que ngrok tourne, accédez à :
- **Dashboard local** : http://localhost:4040
- Vous y verrez toutes les requêtes en temps réel

## Configuration Avancée

### Fichier de configuration ngrok.yml

Créez `C:\Users\VOTRE_USER\.ngrok2\ngrok.yml` :

```yaml
version: "2"
authtoken: VOTRE_TOKEN_ICI
tunnels:
  jenkins:
    proto: http
    addr: 8080
    inspect: true
  jenkins-agent:
    proto: tcp
    addr: 50000
```

### Démarrer avec configuration
```powershell
# Démarrer un tunnel spécifique
ngrok start jenkins

# Démarrer tous les tunnels
ngrok start --all
```

## Sécurité avec Ngrok

### 1. Activer l'authentification
```powershell
ngrok http 8080 --basic-auth="admin:votreMotDePasse"
```

### 2. Restreindre les IPs (Plan payant)
```yaml
tunnels:
  jenkins:
    proto: http
    addr: 8080
    ip_restriction:
      allow_cidrs:
        - 203.0.113.0/24
```

### 3. Webhook verification
Jenkins avec ngrok fonctionne bien avec les webhooks GitHub/GitLab pour déclencher des builds automatiquement.

## Commandes Utiles

```powershell
# Voir la version
ngrok version

# Tester la connexion
ngrok diagnose

# Liste des tunnels actifs
ngrok api tunnels list

# Arrêter ngrok
# Appuyez sur Ctrl+C dans le terminal
```

## Limitations du Plan Gratuit

- ✅ 1 processus ngrok simultané
- ✅ 4 tunnels par processus
- ✅ 40 connexions/minute
- ⚠️ URL aléatoire qui change à chaque redémarrage
- ⚠️ Pas de sous-domaine personnalisé

## Alternative : Ngrok en Arrière-Plan

### Docker Compose avec Ngrok

Ajoutez à votre `docker-compose.yml` :

```yaml
services:
  jenkins:
    # ... configuration existante ...

  ngrok:
    image: ngrok/ngrok:latest
    container_name: ngrok-jenkins
    restart: unless-stopped
    command:
      - "start"
      - "--all"
      - "--config"
      - "/etc/ngrok.yml"
    volumes:
      - ./ngrok.yml:/etc/ngrok.yml
    ports:
      - "4040:4040"  # Dashboard ngrok
    depends_on:
      - jenkins
```

Créez `ngrok.yml` dans votre projet :

```yaml
version: "2"
authtoken: VOTRE_TOKEN_ICI
tunnels:
  jenkins:
    proto: http
    addr: jenkins:8080
```

## Webhook GitHub avec Ngrok

1. Obtenez votre URL ngrok : `https://abc123.ngrok.io`
2. Dans votre repo GitHub → **Settings** → **Webhooks**
3. **Payload URL** : `https://abc123.ngrok.io/github-webhook/`
4. **Content type** : `application/json`
5. **Events** : `Just the push event`

## Troubleshooting

### Erreur : "authtoken required"
```powershell
ngrok config add-authtoken VOTRE_TOKEN
```

### Erreur : "failed to start tunnel"
- Vérifiez que Jenkins est démarré
- Vérifiez que le port 8080 est bien accessible

### Tunnels qui se ferment
- Plan gratuit : tunnels se ferment après 2h d'inactivité
- Solution : Plan payant ou redémarrer ngrok

## Pour Production

⚠️ **Ngrok n'est PAS recommandé pour la production !**

Pour production, utilisez :
- Serveur VPS avec IP fixe
- Reverse proxy (Nginx, Traefik)
- Certificat SSL Let's Encrypt
- Nom de domaine propre
