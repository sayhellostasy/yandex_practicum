pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Запускать автоматически по крону раз в 5 минут
    }
    environment {
        RESOURCE_GROUP = 'jeni-jeni' // Название группы ресурсов Azure
        WEBAPP_NAME = 'jeni-jeni-app' // Название веб-приложения в Azure
        MIDDLEWARE_APP_NAME = 'middleware-jeni-jeni' // Название промежуточного приложения в Azure
        MIDDLEWARE_PROD_NAME = 'middleware-jeni-jeni-prod' // Название промежуточного продукта в Azure
    }
    tools {
        maven 'maven-3.8.1' // Для сборки бэкенда нужен Maven
        jdk 'jdk16' // Java Developer Kit нужной версии
        nodejs 'node16' // NodeJS нужной версии для фронтенда
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                docker run --rm -u root mcr.microsoft.com/azure-cli pip install decorator
                '''
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
                    slackSend channel: '#jeni-jeni', color: 'good', message: "Процесс сборки бэкенда успешно завершен!"
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
                failure {
                    slackSend channel: '#jeni-jeni', color: 'danger', message: "Ошибка в процессе сборки бэкенда!"
                }
            }
        }

        stage('Build frontend') {
            steps {
                dir("frontend") {   
                    sh 'npm install' // Загрузим все сторонние зависимости
                    sh 'npm run build' // Запустим сборку
                }
            }    
            post {
                success {
                    slackSend channel: '#jeni-jeni', color: 'good', message: "Процесс сборки фронтенда успешно завершен!"
                }
                failure {
                    slackSend channel: '#jeni-jeni', color: 'danger', message: "Ошибка в процессе сборки фронтенда!"
                }
            }    
        }

        stage('Deploy to Middleware App') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh '''
                    docker run --rm -u root -v $(pwd):/app -w /app mcr.microsoft.com/azure-cli bash -c '
                    az login --service-principal -u $CLIENT_ID -p $CLIENT_SECRET --tenant $TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $MIDDLEWARE_APP_NAME --src backend/target/your-app.war
                    '
                    '''
                }
            }
        }
        
        stage('Deploy to Middleware Prod') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh '''
                    docker run --rm -u root -v $(pwd):/app -w /app mcr.microsoft.com/azure-cli bash -c '
                    az login --service-principal -u $CLIENT_ID -p $CLIENT_SECRET --tenant $TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $MIDDLEWARE_PROD_NAME --src backend/target/your-app.war
                    '
                    '''
                }
            }
        }
        
        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh '''
                    docker run --rm -u root -v $(pwd):/app -w /app mcr.microsoft.com/azure-cli bash -c '
                    az login --service-principal -u $CLIENT_ID -p $CLIENT_SECRET --tenant $TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $WEBAPP_NAME --src backend/target/your-app.war
                    '
                    '''
                }
            }
        }
    }
}