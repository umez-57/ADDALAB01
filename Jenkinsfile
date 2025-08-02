pipeline {
  agent any

  // pick up your globally-configured JDK & Maven tool installations
  tools {
    jdk 'jdk-17'
    maven 'Maven 3.9'
  }

  environment {
    // change to your Docker Hub (or other) namespace
    DOCKER_REGISTRY = 'docker.io/youruser'
    IMAGE_NAME      = 'inventory-service'
  }

  stages {

    stage('Checkout') {
      steps {
        // grab the root Jenkinsfile + inventory-service/ folder
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        // run mvn clean test inside the submodule
        dir('inventory-service') {
          sh 'mvn clean test'
        }
      }
    }

    stage('Package') {
      steps {
        dir('inventory-service') {
          // skip tests since we already ran them
          sh 'mvn package -DskipTests'
        }
      }
    }

    stage('Docker Build & Push') {
      steps {
        script {
          // tag = docker.io/youruser/inventory-service:<build number>
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
        // update your Kubernetes deployment in namespace=staging
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
      // archive the JAR so you can download it from Jenkins
      archiveArtifacts artifacts: 'inventory-service/target/*.jar', fingerprint: true
    }
    failure {
      // make sure your Jenkins master can send mail or remove this block
      mail to: 'team@example.com',
           subject: "Build ${currentBuild.fullDisplayName} Failed",
           body: "See ${env.BUILD_URL}"
    }
  }
}
