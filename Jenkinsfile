pipeline {
    agent {
        node {
            label 'slave'
        }
    }
    
    triggers {
        pollSCM('H/5 * * * *') // Запускать будем автоматически по крону примерно раз в 5 минут
    }

    environment {
        AZURE_CREDENTIALS = credentials('azure-app') // Указываете ID ваших учетных данных Azure из Jenkins Credentials
        RESOURCE_GROUP = 'jeni-jeni' // Название вашей группы ресурсов Azure
        WEBAPP_NAME = 'jeni-jeni-app' // Название вашего веб-приложения в Azure
        MIDDLEWARE_APP_NAME = 'middleware-jeni-jeni' // Название вашего промежуточного приложения в Azure
        MIDDLEWARE_PROD_NAME = 'middleware-jeni-jeni-prod' // Название вашего промежуточного продукта в Azure
    }

    tools {
        maven 'maven-3.8.1' // Для сборки бэкенда нужен Maven
        jdk 'jdk16' // И Java Developer Kit нужной версии
        nodejs 'node16' // А NodeJS нужен для фронтафффdasa
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'sudo apt install pip'
                sh 'pip install decorator' // Установка недостающего модуля Python
            }
        }
        stage('Build & Test backend') {
            steps {
                dir("backend") { // Переходим в папку backend
                    sh 'mvn package' // Собираем мавеном бэкенд
                }
            }

            post {
                success {
                    slackSend channel: '#jeni-jeni', color: 'good', message: "Процеs сборки бекенда успешно заверше1sн!"
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
        stage('Deploy to Middleware App') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh """
                    az login --service-principal -u \$CLIENT_ID -p \$CLIENT_SECRET --tenant \$TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $MIDDLEWARE_APP_NAME --src backend/target/your-app.war
                    """
                }
            }
        }
        
        stage('Deploy to Middleware Prod') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh """
                    az login --service-principal -u \$CLIENT_ID -p \$CLIENT_SECRET --tenant \$TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $MIDDLEWARE_PROD_NAME --src backend/target/your-app.war
                    """
                }
            }
        }
        
        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh """
                    az login --service-principal -u \$CLIENT_ID -p \$CLIENT_SECRET --tenant \$TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $WEBAPP_NAME --src backend/target/your-app.war
                    """
                }
            }
        }
    }
}