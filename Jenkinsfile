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
                    slackSend channel: '#jeni-jeni', color: 'good', message: "Процеs сборки бекенда успешно завершен!"
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
                failure {
                    slackSend channel: '#jeni-jeni', color: 'danger', message: "Ошибка в процессе сборки бека!"
                }
            }
        }

        stage('Build frontend') {
            steps {
                dir("frontend") {   
                    sh 'npm install' // Для фронта сначала загрузим все сторонние зависимости
                    sh 'npm run build' // Запустим сборку  ЫЫЫЫААААА
                }
            }    
            post {
                success {
                    slackSend channel: '#jeni-jeni', color: 'good', message: "Процеs сборки бекенда успешно завершен!"
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
                failure {
                    slackSend channel: '#jeni-jeni', color: 'danger', message: "Ошибка в процессе сборки бека!"
                }
            }    

            
        }
          stage('Deploy Staging') {
      steps {
            azureWebAppPublish appName: "middleware-staging",
            azureCredentialsId: "azure-app",
            publishType: "file",
            filePath: "**/*.*",
            resourceGroup: "jeni-jeni"
      }
    }
    stage("Approval") {
        steps {
            emailext (
                    attachLog: true,
                    subject: "Waiting for your Approval! Job: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    mimeType: 'text/html',
                    body: """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                                <p>Job has completed dev/test successfully and deployed to Staging! Please go to this link and Approve the build to promote to production. </br><a href='${env.BUILD_URL}/input'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
                    recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
            script {
                def userInput = false
                userInput = input(id: 'Proceed1', message: 'Promote to Production?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this!']])
                echo 'userInput: ' + userInput

                if(userInput == true) {
                    // do action
                } else {
                    // not do action
                    echo "Action was aborted."
                }
            }    
        }  
    }

    stage('Deploy Prod') {
      steps {
          sh 'echo "Deploying to Production"'
          //emailext attachLog: true, body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}, build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}", recipientProviders: [[$class: 'CulpritsRecipientProvider']], subject: "Jenkins Build - ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            azureWebAppPublish appName: "middleware-jeni-jeni-prod",
            azureCredentialsId: "azure-app",
            publishType: "file",
            filePath: "**/*.*",
            resourceGroup: "jeni-jeni"
      }
    }
  }
  post {
        always {
            echo 'Deployed to Production'
            emailext attachLog: true, body: "${currentBuild.currentResult}: Job ${env.JOB_NAME}, build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}", recipientProviders: [[$class: 'CulpritsRecipientProvider']], subject: "Jenkins Build - ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
    }
  }
} 
