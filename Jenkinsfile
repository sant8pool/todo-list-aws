pipeline {
    agent any

    environment {
       PATH = "${HOME}/.local/bin:${env.PATH}"  // Añadido el path de los binarios instalados con --user
       gitCredentialsId = 'acceso_mikel'
    }

    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/sant8pool/todo-list-aws.git', credentialsId: env.gitCredentialsId
                sh 'echo "PYTHONPATH is set to: $PYTHONPATH"'
            }
        }
        
        stage('Static Test') {
            steps {
                script {
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        // Instalar flake8 y bandit
                        //sh 'pip install --user flake8 bandit'
                        // Ejecutar análisis estático
                        sh '''
                        flake8 --exit-zero --format=pylint src/ > flake8.out
                        bandit --exit-zero -r src/ -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                        '''
                    }
                }
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        // Construir la aplicación
                        sh 'sam build'

                        // Desplegar la aplicación en el entorno de Staging usando el archivo de configuración
                        sh 'sam deploy --config-env staging --config-file samconfig.toml --resolve-s3'
                    }        
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                        // Ejecutar pruebas con Pytest
                        sh 'python3 -m pytest --junitxml=report.xml test/integration/todoApiTest.py'
                        junit 'report.xml'
                    }    
                }
            }
        }
        
        stage('Promote') {
            steps {
                script {
                    // Obtener el código más reciente de la rama master
                    git branch: 'master', url: 'https://github.com/sant8pool/todo-list-aws.git', credentialsId: env.gitCredentialsId
        
                    // Hacer checkout de la rama master
                    sh 'git checkout master'
        
                    // Mergear la rama develop en master
                    sh 'git merge develop'
        
                    // Asegurar que las credenciales se pasen correctamente al hacer el git push
                    withCredentials([usernamePassword(credentialsId: env.gitCredentialsId, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASSWORD')]) {
                        sh '''
                            git config --global user.name "Tu Nombre"
                            git config --global user.email "tu-correo@dominio.com"
                            git push https://$GIT_USER:$GIT_PASSWORD@github.com/sant8pool/todo-list-aws.git master
                        '''
                    }    
                }
            }
        }
    }
    
    post {
        always {
            // limpiar
            cleanWs()
        }
    }
}