pipeline {
    agent {
        node {
            label 'slave'
        }
    }
            
    triggers {
        pollSCM('H/5 * * * *') // Запускать будем автоматически по крону примерно раз в 5 минут
    }

    tools {
        maven 'maven-3.8.1' // Для сборки бэкенда нужен Maven
        jdk 'jdk16' // И Java Developer Kit нужной версии
        nodejs 'node16' // А NodeJS нужен для фронтафффdasa
    }

    stages {
        stage('Build & Test backend') {
            steps {
                dir("backend") { // Переходим в папку backend
                    sh 'mvn package' // Собираем мавеном бэкенд
                }
            }

            post {
                success {
                    slackSend channel: '#general', color: 'good', message: "Процесс сборки бекенда успешно завершен!"
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
                failure {
                    slackSend channel: '#general', color: 'danger', message: "Ошибка в процессе сборки бека!"
                }
            }
        }

        stage('Build frontend') {
            steps {
                dir("frontend") {    
                    sh 'npm cache clean --force'
                    sh 'npm install' // Для фронта сначала загрузим все сторонние зависимости
                    sh 'npm run build' // Запустим сборку  ЫЫЫЫААААА
                }

            }
            post {
                success {
                    slackSend channel: '#general', color: 'good', message: "Процесс сборки фронтенда успешно завершен!"
                }
                failure {
                    slackSend channel: '#general', color: 'danger', message: "Ошибка в процессе сборки фронта!"
                }
            }
        }   

        
        stage('Save artifacts') {
            steps {
                archiveArtifacts(artifacts: 'backend/target/sausage-store-0.0.1-SNAPSHOT.jar')
                archiveArtifacts(artifacts: 'frontend/dist/frontend/*')
                slackSend channel: '#general', color: 'good', message: 'артефыаsктsdsaыы сохранены'
            }
        }
    }
}