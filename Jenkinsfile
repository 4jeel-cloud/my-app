pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'ajeeldocker'
    GITHUB_USER    = '4jeel-cloud'
    K8S_REPO       = 'github.com/4jeel-cloud/my-app-k8s.git'
    IMAGE_NAME     = 'my-app'
    K8S_FILE       = 'deployment.yaml'
  }

  stages {
    stage('Set Image Tag') {
      steps {
        script {
          env.IMAGE_TAG = sh(
            script: 'git rev-parse --short HEAD',
            returnStdout: true
          ).trim()
          echo "Image tag: ${env.IMAGE_TAG}"
        }
      }
    }

    stage('Build Image') {
      steps {
        sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${env.IMAGE_TAG} ."
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-credentials',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }

    stage('Update Manifest') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'github-credentials',
          usernameVariable: 'GIT_USER',
          passwordVariable: 'GIT_PASS'
        )]) {
          sh '''
            rm -rf k8s-repo
            git clone https://${GIT_USER}:${GIT_PASS}@${K8S_REPO} k8s-repo
            cd k8s-repo
            sed -i "s|image: .*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|" ${K8S_FILE}
            git config user.email "jenkins@ci.local"
            git config user.name "Jenkins CI"
            git add ${K8S_FILE}
            git diff --cached --quiet && echo "No changes to commit" || git commit -m "Update image to ${IMAGE_TAG}"
            git push origin main
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Pipeline succeeded. Image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    failure {
      echo "Pipeline failed. Check the stage logs above."
    }
  }
}
