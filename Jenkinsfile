pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "rani06/bms:latest"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Rani-06/Book-My-Show.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=BMS \
                    -Dsonar.projectKey=BMS
                    '''
                }
            }
        }

        // QUALITY GATE REMOVE CHESAM
        // ENDUKANTE SOMETIMES PIPELINE WAIT AVUTUNDI

        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app

                rm -rf node_modules package-lock.json

                npm install
                '''
            }
        }

        // OWASP OPTIONAL
        // Jenkins lo DP-Check configure cheyyakapothe skip avvadaniki try-catch use chesam

        stage('OWASP Scan') {
            steps {
                script {
                    try {

                        dependencyCheck(
                            additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit',
                            odcInstallation: 'DP-Check'
                        )

                        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'

                    } catch (Exception e) {

                        echo "OWASP Dependency Check skipped because DP-Check not configured"

                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                trivy fs . > trivyfs.txt
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker-cred') {

                        sh '''
                        docker build --no-cache -t $IMAGE_NAME -f bookmyshow-app/Dockerfile bookmyshow-app

                        docker push $IMAGE_NAME
                        '''
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {

                sh '''
                docker stop bms || true

                docker rm bms || true

                docker run -d --name bms -p 3000:3000 $IMAGE_NAME

                docker ps
                '''
            }
        }
    }

    post {

        always {

            emailext(
                attachLog: true,
                subject: "${currentBuild.result}: Job ${env.JOB_NAME}",
                body: """
Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}
""",
                to: 'YOURMAIL@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
