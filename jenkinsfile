pipeline{
    agnet any
    environment{
        NAME_APP = "juice-shop"
    }

    stages{
        stage('Repositório de Código'){
            steps{
                git url: 'https://github.com/ThViana94/juice-shop',
                    branch: 'master'
                stash 'devsec-repositorio'
            }
        }

        stage('SAST - Horusec') {
            environment{
                // variaveis especificas para o OWASP ZAP
                HORUSEC_API_KEY = credentials('api-key-horusec')
            }
            // Configuração do Agent automation
            agent{ node 'automation'}
            steps{
                sh 'horusec start -a $HORUSEC_API_KEY -u http://192.168.56.60:8000 -p $(WORKSPACE)/ -G true --enable-git-history="true"'
            }
        }

        stage('Build + Executando Container'){
            agent { node 'automation'}
            steps{
                sh "docker-compose up -d"
            }
        }

        stage ('DAST com Golismero'){
            environment{
                APP_URL = "http://192.168.56.60"
            }
            agent { node 'testing'}
            steps{
                sh "/opt/golismero/golismero.py scan $APP_URL -o dast-report-${BUILD_NUMBER}.html"
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'dast-report-${BUILD_NUMBER}.html',
                    reportName: 'DAST REPORT',
                ])
            }
        }

    }

    post{
        always{
            chuckNorris()
        }
        success{
            echo "Pipeline executada com sucesso !!"
        }
        failure{
            echo "Pipeline falhou "
        }
    }
}
