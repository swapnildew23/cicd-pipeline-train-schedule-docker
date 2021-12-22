pipeline {
    agent any
    stages {
        stage('Build Stage') {
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
                    app = docker.build("swap/test-train-schedule")
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            when {
                branch 'master'
            }
            steps {
                script {
                    try {
                        docker.withRegistry('https://hub.docker.com', 'docker-hub') {
                            app.push["${env.BUILD_NUMBER}"]
                            app.push["latest"]
                        }
                    } catch (err) {
                            echo: 'caught error: $err'
                    }
                }
            }
        }
        stage('Deploy to Production Server') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production Server'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'web-server-login', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull swap/test-train-schedule:${env.BUILD_NUMBER}\" "
                        try {
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop test-train-schedule\" "
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm test-train-schedule\" "
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERNAME' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name test-train-schedule -p 8080:8080  -d swap/test-train-schedule:${env.BUILD_NUMBER}\" "
                    }                    
                }  
            }
        }
    }
}
