pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')  // Single credential ID
    DOCKERHUB_IMAGE_PREFIX = "${DOCKERHUB_CREDENTIALS_USR}"
  }

  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/your-username/your-repo.git'
      }
    }

    stage('Docker Login') {
      steps {
        script {
          sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
        }
      }
    }

    stage('Build & Push Backend') {
      steps {
        dir('backend') {
          script {
            sh "docker build -t ${DOCKERHUB_IMAGE_PREFIX}/backend-app ."
            sh "docker push ${DOCKERHUB_IMAGE_PREFIX}/backend-app"
          }
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        dir('frontend') {
          script {
            sh "docker build -t ${DOCKERHUB_IMAGE_PREFIX}/frontend-app ."
            sh "docker push ${DOCKERHUB_IMAGE_PREFIX}/frontend-app"
          }
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent(['ec2-ssh-key']) {
          sh """
          ssh -o StrictHostKeyChecking=no ec2-user@<EC2-IP> << EOF
            docker pull ${DOCKERHUB_IMAGE_PREFIX}/backend-app
            docker pull ${DOCKERHUB_IMAGE_PREFIX}/frontend-app

            docker stop backend || true && docker rm backend || true
            docker stop frontend || true && docker rm frontend || true

            docker run -d --restart unless-stopped -p 4000:4000 --name backend ${DOCKERHUB_IMAGE_PREFIX}/backend-app
            docker run -d --restart unless-stopped -p 80:3000 --name frontend ${DOCKERHUB_IMAGE_PREFIX}/frontend-app
          EOF
          """
        }
      }
    }
  }
}
