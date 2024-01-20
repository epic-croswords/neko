pipeline {
    agent any
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
    stage('checkout-code') {
            steps {
                git branch: 'master', url: 'https://github.com/epic-croswords/neko.git'
            }
        }
    stage('owasp dependancy check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./  --disableNodeAudit ', odcInstallation: 'DC'
               dependencyCheckPublisher pattern: '**/dependancy-check-report.xml'
            }
        }
    
    stage('sonar-qube') {
            steps {
                withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=VirtualBrowser \
                  -Dsonar.projectName=VirtualBrowser '''
                  
            }
        }
    }
    stage('docker build & test') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                    dir('/var/lib/jenkins/workspace/virtual-browser/.docker/google-chrome') {
                    sh "docker build -t manlineroot12/virtualbrowser:v2 ."
                }
                    }
            }
        }
    }
        stage('trivy') {
            steps {
                sh "trivy image manlineroot12/virtualbrowser:v2"
        }
    }
    stage('docker push image') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                dir('/var/lib/jenkins/workspace/virtual-browser/.docker/google-chrome') {
                    echo "Logging into Docker Hub..."
                    sh "docker login -u 'username@username.com' -p 'password@password' https://index.docker.io/v1/"

                    echo "Pushing Docker image to Docker Hub..."
                    sh "docker push manlineroot12/virtualbrowser:v2"
                }
            }
        }
    }
}
    stage('docker-compose') {
        steps {
            sh "docker-compose up -d"
            }
    }
    }
}
