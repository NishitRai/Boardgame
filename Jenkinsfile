pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: '4d7cdeb2-d39c-4d75-af23-ea1a6e59c603', url: 'https://github.com/NishitRai/Boardgame.git'
            }
        }
    
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('File System scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html --scanners vuln,misconfig,secret .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.key=BoardGame \
                    -Dsonar.java.binaries=. 
                    '''
                }
            }
        }

        stage('Sonar Quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }        

        stage('Build and Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-credentials', toolName: 'docker') {
                        sh 'docker build -t nish536/boardgame:latest .'
                    }                    
                }
            }
        }
 
        stage('Docker Image scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html --scanners vuln nish536/boardgame:latest'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-credentials', toolName: 'docker') {
                        sh 'docker push nish536/boardgame:latest'
                    }                    
                }
            }
        }
        
    }
}
