pipeline {
  agent any

  environment {
    JFROG_HOST = "trial227jwz.jfrog.io"
    DOCKER_REPO = "docker-local"
    IMAGE_NAME  = "adon-petclinic"
    IMAGE_TAG   = "1.0.${BUILD_NUMBER}"
    FULL_IMAGE  = "${JFROG_HOST}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Build + Test') {
      steps { sh '''
      ./mvnw clean test \
        -Dspring.docker.compose.skip.in-tests=true \
        -Dtest=!PostgresIntegrationTests
    ''' }
    }

    stage('Package') {
      steps { sh './mvnw -DskipTests package' }
    }

    stage('Build Docker Image') {
      steps { sh 'docker build -t adon-petclinic:latest .' }
    }

    stage('Login to JFrog') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'jfrog-creds',
                                          usernameVariable: 'JFROG_USER',
                                          passwordVariable: 'JFROG_TOKEN')]) {
          sh '''
            echo "$JFROG_TOKEN" | docker login ${JFROG_HOST} -u "$JFROG_USER" --password-stdin
          '''
        }
      }
    }

    stage('Tag + Push') {
      steps {
        sh '''
          docker tag adon-petclinic:latest ${FULL_IMAGE}
          docker push ${FULL_IMAGE}
        '''
      }
    }
  }
}
