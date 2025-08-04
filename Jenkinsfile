pipeline {
    agent any

    tools {
        jdk   'jdk-17'
        maven 'Maven 3.9'
    }

    /* ---------- GLOBAL ENV ---------- */
    environment {
        DOCKER_REGISTRY_URL  = 'https://index.docker.io/v1/'
        DOCKER_REGISTRY_CRED = 'dockerhub-creds'      // your Jenkins Credentials ID
        DOCKER_NAMESPACE     = 'umez57'               // Docker Hub user/org
        IMAGE_NAME           = 'inventory-service'
    }

    /* ---------- STAGES ---------- */
    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                dir('inventory-service') {
                    sh './mvnw clean test'
                }
            }
        }

        stage('Package') {
            steps {
                dir('inventory-service') {
                    sh './mvnw package -DskipTests'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    dir('inventory-service') {

                        // 1) Build the image
                        def img = docker.build(
                            "${DOCKER_NAMESPACE}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        )

                        // 2) Login & Push both "latest" and build-number tags
                        docker.withRegistry(
                            env.DOCKER_REGISTRY_URL,
                            env.DOCKER_REGISTRY_CRED
                        ) {
                            img.push()                      // pushes :latest
                            img.push("${env.BUILD_NUMBER}") // pushes :<build_number>
                        }
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                expression {
                    // only run on the 'main' branch
                    return env.BRANCH_NAME == 'main'
                }
            }
            steps {
                sh """
                  kubectl set image deployment/${IMAGE_NAME} \
                    ${IMAGE_NAME}=${DOCKER_NAMESPACE}/${IMAGE_NAME}:${env.BUILD_NUMBER} \
                    --namespace=staging
                """
            }
        }
    }

    /* ---------- POST ACTIONS ---------- */
    post {
        always {
            archiveArtifacts artifacts: 'inventory-service/target/*.jar', fingerprint: true
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Build ${currentBuild.fullDisplayName} Failed",
                 body: "See ${env.BUILD_URL}"
        }
    }
}
