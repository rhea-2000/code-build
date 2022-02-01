pipeline {
    agent any
    environment {
        registry = "993745358053.dkr.ecr.us-east-1.amazonaws.com/my-docker-repo"
    }
    stages {
        stage ('Checkout') {
            steps {
              checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/rhea-2000/myPython-Docker-Repo.git']]])
            }
        }
        stage ('Build') {
            steps {
                  script {
                       dockerImage = docker.build registry
                     //'docker build -t my-docker-repo .'
                     //'docker tag my-docker-repo:latest 993745358053.dkr.ecr.us-east-1.amazonaws.com/my-docker-repo:latest'
                  }
            }    
        }
        stage ('Push') {
            steps {
                script {
                     sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 993745358053.dkr.ecr.us-east-1.amazonaws.com"
                     sh "docker push 993745358053.dkr.ecr.us-east-1.amazonaws.com/my-docker-repo:latest"
                }
            }
        }
        stage ('Run') {
            steps {
                script {
                    sh "docker run -d -p 8080:8070 --rm --name my-python-container 993745358053.dkr.ecr.us-east-1.amazonaws.com/my-docker-repo:latest"
                }
            }
        }
    }
}
