pipeline {

    agent any

    environment {
        COMPOSE_DEV = "docker-compose -f docker-compose.angular.dev.yml"
        COMPOSE_PROD = "docker-compose -f docker-compose.angular.prod.yml"
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage("Clean Workspace"){
            steps{
                deleteDir()
            }
        }

        stage("Checkout SCM"){
            steps{
                checkout scm
            }
        }

        stage("Stop previous Angular"){
            steps{
                sh'''
                echo "Parando continaer que usam porta 4200..."
                docker ps -q --filter "publish=4200" | xargs -r docker stop
                docker ps -aq --filter "publish=4200" | xargs -r docker rm
                '''
            }
        }

        stage("Build Angular DEV"){
            steps{
                sh'''
                echo "Subindo Angular DEV..."
                ${COMPOSE_DEV} down || true
                docker system prune -f || true
                ${COMPOSE_DEV} up -d --build --remove-orphans
                '''
            }
        }

        stage("Test Angular DEV"){
            steps{
                sh'''
                echo "Aguardando Angular subir..."
                sleep 50
                
                echo "Testando Angular DEV..."
                curl -f http://angular-dev:4200 || exit 1
                '''
            }
        }

        stage("Aprovação PROD"){
            steps{
                input message: 'Deseja promover Angular para PROD?'
            }
        }

        stage("Deploy Angular PROD"){
            steps{
                sh'''
                echo "Subindo Angular PROD..."
                ${COMPOSE_PROD} down || true
                ${COMPOSE_PROD} up -d --build
                '''
            }
        }
    }

    post{
        
        success {
            echo '✅ Frontend deployado com sucesso!'
        }
        failure {
            echo '❌ Falha no pipeline do frontend!'
        }
    }

}