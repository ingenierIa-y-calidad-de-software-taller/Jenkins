pipeline {
    agent any
    tools {
        maven 'maven'
    }

    environment {
        PROJECT_DIR = 'ci-taller-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/ingenierIa-y-calidad-de-software-taller/Taller-Demo-CICD.git', branch: 'main'
            }
        }
/*
        stage('Build') {
            steps {
                dir("${env.PROJECT_DIR}") {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Test') {
            steps {
                dir("${env.PROJECT_DIR}") {
                    sh 'mvn test'
                }
            }
        }
  */  
        stage('Build y Test') {
            steps {
                dir("${env.PROJECT_DIR}") {
                    sh 'mvn clean package'
                }
            }
        }

    /*
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    dir('ci-taller-demo') {
                        sh '''
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=ingenierIa-y-calidad-de-software-taller_Taller-Demo-CICD \
                    -Dsonar.token=${SONAR_TOKEN} \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -X
                    '''
                    }
                }
            }
        }

    */
         stage('Confirmar despliegue') {
            steps {
                script {
                    def userInput = input(
                        id: 'DespliegueConfirmacion',
                        message: '¿Deseas desplegar a producción?',
                        parameters: [
                            choice(choices: ['Sí', 'No'], description: 'Selecciona una opción', name: 'Desplegar')
                        ]
                    )
                    env.DEPLOY = (userInput == 'Sí') ? 'true' : 'false'
                    }
            }
         }


    stage('Deploy on Render') {
        when {
            expression { env.DEPLOY == 'true' }
        }
        steps {
            script {
                withCredentials([
                    string(credentialsId: 'RENDER_API_KEY', variable: 'RENDER_API_KEY'),
                    string(credentialsId: 'RENDER_SERVICE_ID', variable: 'RENDER_SERVICE_ID')
                ]) {
                    sh '''
                        curl -X POST https://api.render.com/deploy/srv-$RENDER_SERVICE_ID \
                            -H "Authorization: Bearer $RENDER_API_KEY" \
                            -H "Accept: application/json"
                    '''
                }
            }
        }
    }

}

    post {
        always {
            script {
                sh """
                    echo "==== Prueba de humo (DemoApplicationTests) ====" > test_result_summary.txt
                    if [ -f ${env.PROJECT_DIR}/target/surefire-reports/com.example.demo.DemoApplicationTests.txt ]; then
                        grep "Tests run:" ${env.PROJECT_DIR}/target/surefire-reports/com.example.demo.DemoApplicationTests.txt >> test_result_summary.txt
                    else
                        echo "No se encontraron resultados." >> test_result_summary.txt
                    fi

                    echo "" >> test_result_summary.txt
                    echo "==== Resultado HelloWorldControllerTest ====" >> test_result_summary.txt
                    if [ -f ${env.PROJECT_DIR}/target/surefire-reports/com.example.demo.controllers.HelloWorldControllerTest.txt ]; then
                        grep "Tests run:" ${env.PROJECT_DIR}/target/surefire-reports/com.example.demo.controllers.HelloWorldControllerTest.txt >> test_result_summary.txt
                    else
                        echo "No se encontraron resultados." >> test_result_summary.txt
                    fi
                """
            }
        }

        success {
            withCredentials([
                string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'BOT_TOKEN'),
                string(credentialsId: 'TELEGRAM_GROUP_CHAT_ID', variable: 'CHAT_ID')
            ]) {
                script {
                    def testSummary = readFile('test_result_summary.txt').trim()
                    sh """
                        curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \\
                        -d chat_id=${CHAT_ID} \\
                        -d text="✅  Éxito: El pipeline se ha completado correctamente.\n ${testSummary}"
                    """
                }
            }
        }

        failure {
            withCredentials([
                string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'BOT_TOKEN'),
                string(credentialsId: 'TELEGRAM_CHAT_ID', variable: 'CHAT_ID')
            ]) {
                script {
                    def testSummary = fileExists('test_result_summary.txt') ? readFile('test_result_summary.txt').trim() : "Sin resultados."
                    sh """
                        curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \\
                        -d chat_id=${CHAT_ID} \\
                        -d text="❌  Error: EL pipeline ha fallado.\n ${testSummary}"
                    """
                }
            }
        }

        aborted {
                withCredentials([
                string(credentialsId: 'TELEGRAM_BOT_TOKEN', variable: 'BOT_TOKEN'),
                string(credentialsId: 'TELEGRAM_CHAT_ID', variable: 'CHAT_ID')
            ]) {
                    script {
                        def testSummary = fileExists('test_result_summary.txt') ? readFile('test_result_summary.txt').trim() : "Sin resultados."
                    sh """
                        curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \\
                        -d chat_id=${CHAT_ID} \\
                        -d text="⚠️  Deploy abortado.\n ${testSummary}"
                    """
                }
            }
        }
    }

}
