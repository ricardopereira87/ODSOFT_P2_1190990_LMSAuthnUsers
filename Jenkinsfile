pipeline {
    agent any

    environment {

        //MAVEN_HOME = '/opt/homebrew' // (MacOS)
        MAVEN_HOME = '/usr/share/maven' // (Ubuntu)

        GIT_REPO_URL = 'https://github.com/ricardopereira87/ODSOFT_P2_1190990_LMSAuthnUsers'  // Your Git repository URL
        GIT_BRANCH = 'main'  // Specify the branch to check out
        //CREDENTIALS_ID = 'x'  // Credentials ID for authentication

        SERVER_PORT = '2226'

        IMAGE_NAME = 'lmsbooks'
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('Debug Environment') {
            steps {
                sh 'env'
            }
        }


        stage('Check Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Checkout') {
            steps {
                // Step to clone the Git repository
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO_URL}"//,
                    //credentialsId: "${CREDENTIALS_ID}"
            }
        }

        stage('Clean') {
            steps {
                script {
                    sh """
                        mvn clean install
                    """
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    sh "mvn validate"
                }
            }
        }

        stage('Compile') {
            steps {
                script {
                    sh "mvn compile"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Run tests
                    withEnv(["PATH+JDK=${JAVA_HOME}/bin"]) {
                        sh "mvn test"
                    }
                }
            }
        }

        stage('Package') {
            steps {
                script {
                    // Package the application
                    sh "mvn package"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    sh """

                        if ! docker ps --filter "name=rabbitmq_in_lms_network" --format '{{.Names}}' | grep -q rabbitmq_in_lms_network; then
                          docker compose -f docker-compose-rabbitmq.yml up -d
                        else
                          echo "RabbitMQ container already running."
                        fi

                        if ! docker ps --filter "name=postgres_in_lms_network" --format '{{.Names}}' | grep -q postgres_in_lms_network; then
                          docker compose -f docker-compose-postgres.yml up -d
                        else
                          echo "Postgres container already running."
                        fi


                        docker compose -f docker-compose.yml up -d --force-recreate

                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }

////if containers are to be removed
//     post {
//         always {
//             echo 'Cleaning up...'
//             sh """
//                 docker compose -f docker-compose.yml down || true
//             """
//         }
//     }

}