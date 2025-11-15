pipeline {
    agent {
        docker {
            image 'bash:latest'
            // Pas besoin de monter de volume, Jenkins gère automatiquement le workspace
        }
    }
    
    parameters {
        string(name: 'N', defaultValue: '10', description: 'Nombre de termes de la suite de Fibonacci')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "=== Récupération du code depuis Git ==="
                checkout scm
            }
        }
        
        stage('Vérification de l\'environnement') {
            steps {
                echo "=== Informations de l'environnement ==="
                echo "Workspace: ${WORKSPACE}"
                echo "Nombre de termes Fibonacci: ${params.N}"
                sh 'echo "Agent Docker: $(hostname)"'
                sh 'echo "Bash version: $(bash --version | head -n1)"'
                sh 'pwd'
                sh 'ls -la'
            }
        }
        
        stage('Préparation du script') {
            steps {
                echo "Vérification et préparation du script fibonacci.sh"
                sh '''
                    if [ -f "scripts/fibonacci.sh" ]; then
                        echo "✓ Script fibonacci.sh trouvé"
                        chmod +x scripts/fibonacci.sh
                    else
                        echo "✗ ERREUR: fibonacci.sh non trouvé"
                        echo "Contenu du workspace:"
                        ls -laR
                        exit 1
                    fi
                '''
            }
        }
        
        stage('Exécution Fibonacci') {
            steps {
                script {
                    echo "=== Début du calcul de Fibonacci ==="
                    sh "./scripts/fibonacci.sh ${params.N}"
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