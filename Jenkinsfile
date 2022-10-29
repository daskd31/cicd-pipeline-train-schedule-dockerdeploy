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
        stage('Docker build Image') {
            when{
                branch 'master'   
            }
            steps {
                script {
                    app = docker.build("kanishka2022/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'    
                    }
                }
            }
        }
        stage('Push Docker Image'){
             when{
                branch 'master'   
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com','docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")                        
                    }
                }
            }
        }
        
        stage('Deploy to production') {
             when{
                branch 'master'   
            }
            steps {
                input 'Deploy to production ?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '8dJ-ezS_' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull kanishka2022/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '8dJ-ezS_' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '8dJ-ezS_' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '8dJ-ezS_' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d kanishka2022/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
