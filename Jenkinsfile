pipeline {
  agent any

  environment {
    DOCKERHUB_USERNAME = credentials('dockerhub-username')  // Create Jenkins secret
    DOCKERHUB_PASSWORD = credentials('dockerhub-password')
  }

  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/purveshpeche/my-app.git'
      }
    }

    stage('Build & Push Backend') {
      steps {
        dir('backend') {
          script {
            sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
            sh 'docker build -t $DOCKERHUB_USERNAME/backend-app .'
            sh 'docker push $DOCKERHUB_USERNAME/backend-app'
          }
        }
      }
    }

    stage('Build & Push Frontend') {
      steps {
        dir('frontend') {
          script {
            sh 'docker build -t $DOCKERHUB_USERNAME/frontend-app .'
            sh 'docker push $DOCKERHUB_USERNAME/frontend-app'
          }
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent(['ec2-ssh-key']) {
          sh '''
          ssh -o StrictHostKeyChecking=no ec2-user@<EC2-IP> << 'EOF'
            sudo docker pull $DOCKERHUB_USERNAME/backend-app
            sudo docker pull $DOCKERHUB_USERNAME/frontend-app

            sudo docker stop backend || true && sudo docker rm backend || true
            sudo docker stop frontend || true && sudo docker rm frontend || true

            sudo docker run -d --restart unless-stopped -p 4000:4000 --name backend $DOCKERHUB_USERNAME/backend-app
            sudo docker run -d --restart unless-stopped -p 80:3000 --name frontend $DOCKERHUB_USERNAME/frontend-app
          EOF
          '''
        }
      }
    }
  }
}
