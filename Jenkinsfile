pipeline {
  agent any

  environment {
    DOCKER_BUILDKIT = '0'
    AWS_REGION = 'us-west-2'
    EKS_CLUSTER_NAME = 'praveen-ecomm-application'
    AWS_ACCESS_KEY_ID = 'Praveen-AWS_ACCESS_KEY_ID'
    AWS_SECRET_ACCESS_KEY = 'PRAVEEN_AWS_SECRET_ACCESS_KEY'
  }

  stages {

    stage('Checkout Code') {
      steps {
        git url: 'https://github.com/iampraveen6/E-CommerceStore.git', branch: 'main'
      }
    }

    stage('Build and Push Docker Images') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'Praveen-docker', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {

            def services = [
              'user-service',
              'product-service',
              'cart-service',
              'order-service'
            ]

            services.each { service ->
              dir("backend/${service}") {
                sh """
                  
                  docker build --no-cache -t ${DOCKERHUB_USER}/${service}:latest .
                  echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
                  docker push ${DOCKERHUB_USER}/${service}:latest
                """
              }
            }

          dir("frontend") {
            sh """
              docker build -t ${DOCKERHUB_USER}/frontend:latest .
              echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
              docker push ${DOCKERHUB_USER}/frontend:latest
            """
            }
          }
        }
      }
    }

    stage('Update Kubeconfig') {
      steps {
        sh """
            aws configure set aws_access_key_id ${Praveen-AWS_ACCESS_KEY_ID}
            aws configure set aws_secret_access_key ${PRAVEEN_AWS_SECRET_ACCESS_KEY}
            aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER_NAME}
            aws sts get-caller-identity
            kubectl config view 
            """
      }
    }

    stage('Deploy to EKS') {
      steps {
        sh "kubectl apply -f k8s/"
      }
    }
  }

  post {
    success {
      echo "✅ Deployment to EKS successful!"
    }
    failure {
      echo "❌ Deployment failed. Check logs."
    }
  }
}