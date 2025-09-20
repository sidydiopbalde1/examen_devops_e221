pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
           steps {

                       sh './mvnw clean package'

               }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def appName = 'demo' // Nom de l'application
                        def branchName = env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'latest'
                        def safeTag = branchName.replaceAll('[^A-Za-z0-9._-]', '-')
                        def dockerImage = "${DOCKER_USER}/${appName}:${safeTag}"

                        sh """
                            set -e
                            echo "Building Docker image e: ${dockerImage}"
                            docker build -t "${dockerImage}" .

                            echo "Logging into Docker Hub..."
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin

                            echo "Pushing Docker image: ${dockerImage}"
                            docker push "${dockerImage}"
                        """
                    }
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                withCredentials([string(credentialsId: 'render-hook-java', variable: 'RENDER_HOOK_URL')]) {
                    sh 'curl -X POST "$RENDER_HOOK_URL"'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}