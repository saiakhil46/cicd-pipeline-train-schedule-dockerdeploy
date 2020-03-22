pipeline {
    agent any
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
                    sh "sudo docker build -t saiakhil46/train-schedule ."
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
              withCredentials([usernamePassword(credentialsId: 'docker_hub_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
              sh 'sudo docker login -u $USERNAME -p $PASSWORD'
              sh 'sudo docker tag saiakhil46/train-schedule saiakhil46/train-schedule:${env.BUILD_NUMBER}'
              sh 'sudo docker push saiakhil46/train-schedule:${env.BUILD_NUMBER}'
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
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"sudo docker pull saiakhil46/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"sudo docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"sudo docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"sudo docker run --restart always --name train-schedule -p 8080:8080 -d saiakhil46/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
