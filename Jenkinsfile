pipeline {
    agent {
        node {
            label 'slave'
        }
    }
    
     
    }
    steps{
        tools {
        nodejs = 'node16'
        jdk = 'jdk16'
        maven = 'maven-3.8.1'
        }
    }    
    post {
        success {
            slackSend channel: '#general', color: 'good', message:
                "началась новая сборка, tools will be instaled !"
        }
        failure {
            slackSend channel: '#general', color: 'danger', message:
                "началась новая сборка, instaled tools eroor, check logs !!"
        }
    }
        
    stages {
        stage (' build + test backend'){
            steps {
                dir(backend){
                    sh 'mvn package'
                }
            }
            
            post {
                success {
                    slackSend channel: '#general', color: 'good', message: "Процесс сборки бекенда успешно завершен!"
                    junit 'backend/target/surefire-reports/**/*.xml'
                }
                failure {
                    slackSend channel: '#general', color: 'danger', message: "Ошибка в процессе сборки бека!"
                }
            }
        }
        stage ('build frontend'){
            steps {
                dir(frontend){
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
            post {
                success{
                    slackSend channel: '#general', color: 'good', message: "Процесс сборки фронтенда успешно завершен!"
                }
                failure {
                    slackSend channel: '#general', color: 'danger', message: "Ошибка в процессе сборки фронтенда!"
                }
            }
        }
    }
}                  



