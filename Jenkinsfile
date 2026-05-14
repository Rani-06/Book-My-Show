pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                cd bookmyshow-app
                npm install
                '''
            }
        }

        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit',
                odcInstallation: 'DP-Check'

                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {

                script {

                    withDockerRegistry(credentialsId: 'docker-cred') {

                        sh '''
                        docker build -t rani06/bms:latest -f bookmyshow-app/Dockerfile bookmyshow-app

                        docker push rani06/bms:latest
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

                docker run -d --name bms -p 3000:3000 rani06/bms:latest
                '''
            }
        }
    }

    post {

        always {

            emailext attachLog: true,
            subject: "${currentBuild.result} : Job ${env.JOB_NAME}",
            body: """
            Project: ${env.JOB_NAME}
            Build Number: ${env.BUILD_NUMBER}
            Build URL: ${env.BUILD_URL}
            """,
            to: 'YOURMAIL@gmail.com',
            attachmentsPattern: 'trivyfs.txt'
        }
    }
}
