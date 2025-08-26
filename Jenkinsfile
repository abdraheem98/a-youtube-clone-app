pipeline{
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage ('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage ('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/abdraheem98/a-youtube-clone-app.git'
            }
        }
        stage ('Sonarqube-Analysis') {
            steps {
            withSonarQubeEnv('sonar-server') {
                sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Youtube-CICD \
                    -Dsonar.projectKey=Youtube-CICD
                '''}
            }
        }
        stage ('Quality Gate Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonartoken'
                }
            }

        }

        stage ('NPM Build') {
            steps {
                sh 'npm install'
            }
        }

        stage ('Docker Build and Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t youtube-clone ."
                        sh "docker tag youtube-clone abdraheem98/youtube-clone:latest"
                        sh " docker push abdraheem98/youtube-clone:latest"
                    }
                }
            }
        }
        stage ('Deploy') {
            steps {
                script {
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            sh 'kubectl delete --all pods'
                            sh 'kubectl apply -f deployment.yml'
                            sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
    }
}
