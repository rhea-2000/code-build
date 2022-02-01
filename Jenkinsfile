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
                     'docker run -d -p 8090:8070 --rm --name my-python-container 993745358053.dkr.ecr.us-east-1.amazonaws.com/my-docker-repo:latest'
                }
            }
        }
        stage('Deploy') {
            steps {
                 withCredentials([string(credentialsId: 'AWS_EXECUTION_ROL_SECRET', variable: 'AWS_ECS_EXECUTION_ROL'),string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
                 script {
                     updateContainerDefinitionJsonWithImageVersion()
                      sh ("/usr/local/bin/aws ecs register-task-definition --region ${us-east-1} --family ${python-app-demo} --execution-role-arn ${AWS_ECS_EXECUTION_ROL} --requires-compatibilities ${FARGATE} --network-mode ${awsvpc} --cpu ${256} --memory ${512} --container-definitions file://${./ecs/container-definition-update-image.json}")
                     def taskRevision = sh(script: "/usr/local/bin/aws ecs describe-task-definition --task-definition ${python-demo} | egrep \"revision\" | tr \"/\" \" \" | awk '{print \$2}' | sed 's/\"\$//'", returnStdout: true)
                      sh ("/usr/local/bin/aws ecs update-service --cluster ${python-app} --service ${python-app-service} --task-definition ${python-app-demo}:${taskRevision}")
                   }
             }
         }
     }
 }
}
