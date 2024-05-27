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
        RESOURCE_GROUP = 'jeni-jeni' // Название группы ресурсов Azure
        WEBAPP_NAME = 'jeni-jeni-app' // Название веб-приложения в Azure
        MIDDLEWARE_APP_NAME = 'middleware-jeni-jeni' // Название промежуточног
        AZ_CLI_HOME = "${env.WORKSPACE}/.azure-cli" // Директория для установки Azure CLI
        PATH = "${env.AZ_CLI_HOME}/bin:${env.PATH}" // Добавляем Azure CLI в PATH
    }
    tools {
        maven 'maven-3.8.1' // Для сборки бэкенда нужен Maven
        jdk 'jdk16' // И Java Developer Kit нужной версии
        nodejs 'node16' // А NodeJS нужен для фронтафффdasa
    }

    stages {
        stage('Install Azure CLI') {
            steps {
                sh '''
                # Убедитесь, что pip установлен
                if ! command -v pip &> /dev/null; then
                    echo "pip не установлен. Устанавливаем pip..."
                    curl -sS https://bootstrap.pypa.io/get-pip.py | python3
                fi

                # Создаем директорию для установки Azure CLI
                mkdir -p ${AZ_CLI_HOME}

                # Устанавливаем Azure CLI в пользовательское окружение
                pip install --user azure-cli

                # Проверяем установку
                az --version
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
                    slackSend channel: '#jeni-jeni', color: 'good', message: "Прsоцеs сборки бекенда успешно заверsшен!"
                    junit 'backend/target/surefire-reports/**/*.xml' // Передадим результаты тестов в Jenkins
                }
                failure {
                    slackSend channel: '#jeni-jeni', color: 'danger', message: "Ошибка в про7цессе сборки бека!"
                }
            }    

            
        }
        stage('Deploy Staging') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: 'azure-app', subscriptionIdVariable: 'SUBSCRIPTION_ID', clientIdVariable: 'CLIENT_ID', clientSecretVariable: 'CLIENT_SECRET', tenantIdVariable: 'TENANT_ID')]) {
                    sh '''
                    az login --service-principal -u $CLIENT_ID -p $CLIENT_SECRET --tenant $TENANT_ID
                    az webapp deployment source config-zip -g $RESOURCE_GROUP -n $MIDDLEWARE_APP_NAME --src backend/target/
                    '''
                }
            }
        }
    }
}      