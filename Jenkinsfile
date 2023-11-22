pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "ldfoskey007/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    script {
                        sh 'cp ./train-schedule-kube-canary.yml /tmp'
                        sh 'kubectl create namespace train-schedule-canary'
                        sh 'kubectl apply -f /tmp/train-schedule-kube-canary.yml && rm /tmp/train-schedule-kube-canary.yml'
                 }
              }
           }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    script {
                        sh 'kubectl delete namespace train-schedule-canary'
                        sh 'cp ./train-schedule-kube.yml /tmp'
                        sh 'kubectl create namespace train-schedule'
                        sh 'kubectl apply -f /tmp/train-schedule-kube.yml && rm /tmp/train-schedule-kube.yml'
                  }
               }
            }
        }
    }
}
