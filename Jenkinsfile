pipeline {
    agent {
        docker {
            image 'bash:latest'
            // Monter le projet depuis le volume Jenkins
            args '-v /workspace/03-jenkins-docker:/project'
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
                sh 'ls -la /project/scripts/ || echo "Répertoire /project/scripts non trouvé"'
            }
        }
        
        stage('Préparation du script') {
            steps {
                echo "Vérification et préparation du script fibonacci.sh"
                sh '''
                    if [ -f "/project/scripts/fibonacci.sh" ]; then
                        echo "✓ Script fibonacci.sh trouvé"
                        chmod +x /project/scripts/fibonacci.sh
                    else
                        echo "✗ ERREUR: fibonacci.sh non trouvé"
                        echo "Contenu de /project:"
                        ls -la /project/ || echo "Répertoire /project non accessible"
                        exit 1
                    fi
                '''
            }
        }
        
        stage('Exécution Fibonacci') {
            steps {
                script {
                    echo "=== Début du calcul de Fibonacci ==="
                    sh "/project/scripts/fibonacci.sh ${params.N}"
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