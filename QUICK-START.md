# 🚀 Guide de Démarrage Rapide - Fibonacci Docker Agent

## Étape 1 : Monter le projet dans Jenkins

Ajoutez cette ligne dans votre `docker-compose.yml` existant (section `volumes`) :

```yaml
- //c/Formation/Learning-2026/Jenkins/03-jenkins-docker:/workspace/03-jenkins-docker
```

Puis redémarrez Jenkins :
```powershell
docker-compose restart jenkins
```

## Étape 2 : Créer le Pipeline

1. Ouvrez Jenkins : **http://localhost:8080**
2. Cliquez sur **Nouvel Item**
3. Nom : `Fibonacci-Docker-Agent`
4. Type : **Pipeline**
5. Dans la configuration Pipeline, collez ceci :

```groovy
pipeline {
    agent {
        docker {
            image 'bash:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    parameters {
        string(name: 'N', defaultValue: '10', description: 'Nombre de termes de la suite de Fibonacci')
    }
    
    stages {
        stage('Vérification de l\'environnement') {
            steps {
                echo "=== Informations de l'environnement ==="
                echo "Workspace: ${WORKSPACE}"
                echo "Nombre de termes Fibonacci: ${params.N}"
                sh 'echo "Agent Docker: $(hostname)"'
                sh 'echo "Bash version: $(bash --version | head -n1)"'
            }
        }
        
        stage('Préparation du script') {
            steps {
                echo "Vérification et préparation du script fibonacci.sh"
                sh '''
                    if [ -f "${WORKSPACE}/scripts/fibonacci.sh" ]; then
                        echo "✓ Script fibonacci.sh trouvé dans scripts/"
                        chmod +x "${WORKSPACE}/scripts/fibonacci.sh"
                    else
                        echo "✗ ERREUR: fibonacci.sh non trouvé dans ${WORKSPACE}/scripts/"
                        exit 1
                    fi
                '''
            }
        }
        
        stage('Exécution Fibonacci') {
            steps {
                script {
                    echo "=== Début du calcul de Fibonacci ==="
                    sh "${WORKSPACE}/scripts/fibonacci.sh ${params.N}"
                    echo "=== Fin du calcul ==="
                }
            }
        }
    }
    
    post {
        success {
            echo '=========================================='
            echo '✓ Pipeline exécuté avec succès!'
            echo "✓ ${params.N} termes de Fibonacci calculés"
            echo '=========================================='
        }
        failure {
            echo '=========================================='
            echo '✗ Le pipeline a échoué.'
            echo 'Vérifiez les logs ci-dessus pour plus de détails'
            echo '=========================================='
        }
        always {
            echo 'Nettoyage du workspace terminé.'
        }
    }
}
```

6. Cliquez sur **Enregistrer**

## Étape 3 : Lancer le Build

1. Cliquez sur **Build with Parameters**
2. Entrez un nombre (ex: 15)
3. Cliquez sur **Build**
4. Observez la console Jenkins !

## ✅ C'est tout !

Votre pipeline va :
- Créer un agent Docker avec l'image `bash:latest`
- Exécuter votre script `fibonacci.sh`
- Afficher les résultats
- Nettoyer automatiquement

---

## 📝 Notes Importantes

- **Première exécution** : Docker va télécharger l'image `bash:latest` (quelques secondes)
- **Exécutions suivantes** : Très rapide car l'image est déjà en cache
- **Paramètre N** : Vous pouvez calculer autant de termes que vous voulez !

## 🔍 Vérifier que tout fonctionne

```powershell
# Vérifier que Jenkins tourne
docker ps | Select-String jenkins

# Voir les logs Jenkins
docker logs -f jenkins

# Vérifier que Docker fonctionne dans Jenkins
docker exec jenkins docker ps
```
