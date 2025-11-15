pipeline {
    agent {
        docker {
            image 'bash:latest'
            // Le socket Docker est déjà monté dans votre Jenkins controller
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