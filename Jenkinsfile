pipeline {
    agent any
    
    
    environment {
        SCANNER_HOME= tool 'sonarqube-scanner'
    }

    stages {
        stage('Git') {
            steps {
                git 'https://github.com/adikesavanaidug2404/boardgame.git'
            }
        }
        stage('Maven') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Trivy File Scan') {
            steps {
                sh'trivy fs > trivy-report.txt'
            }
        }
        stage('S3 Bucket Backup') {
            steps {
                sh 'aws s3 cp /var/lib/jenkins/workspace/boardgame/ s3://myprojectsbucker/boardgamefiles/ --recursive'
            }
        }
        stage('Docker Build&Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t adikesavanaidug2404/boardgame:v1 .'
                        
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image adikesavanaidug2404/boardgame:v1 > trivy-report.txt'
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push adikesavanaidug2404/boardgame:v1'
                        
                    }
                }
            }
        }
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker run -d -p 3000:3000 adikesavanaidug2404/boardgame:v1'
                        
                    }
                }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.12.59:6443') {
                    sh 'kubectl apply -f deployment-service.yaml -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}

