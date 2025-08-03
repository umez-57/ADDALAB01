pipeline {
  agent any

  tools {
    jdk 'jdk-17'
    maven 'Maven 3.9'
  }

  environment {
    DOCKER_REGISTRY = 'docker.io/youruser'
    IMAGE_NAME      = 'inventory-service'
  }

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
          def tag = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
          dir('inventory-service') {
            sh "docker build -t ${tag} ."
          }
          sh "docker push ${tag}"
        }
      }
    }

    stage('Deploy to Staging') {
      when { branch 'main' }
      steps {
        sh """
          kubectl set image deployment/${IMAGE_NAME} \
            ${IMAGE_NAME}=${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER} \
            --namespace=staging
        """
      }
    }
  }

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
