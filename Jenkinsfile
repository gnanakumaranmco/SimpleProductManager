pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    ECR_REPO = "522814733729.dkr.ecr.ap-south-1.amazonaws.com/static-web-app-repo"
    IMAGE_TAG = "latest"
  }

  stages {
    stage('Clone Code') {
      steps {
        git branch: 'main', url: 'https://github.com/gnanakumaranmco/SimpleProductManager.git'
      }
    }

    stage('Terraform Init & Apply') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: 'aws_thegk_cred',
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          sh '''
            cd terraform
            terraform init
            terraform apply -auto-approve
          '''
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${ECR_REPO}:${IMAGE_TAG}")
        }
      }
    }

    stage('Login to ECR') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: 'aws_thegk_cred',
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          sh '''
            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
          '''
        }
      }
    }

    stage('Push Image to ECR') {
      steps {
        script {
          docker.withRegistry("https://${ECR_REPO}", 'aws_thegk_cred') {
            sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
          }
        }
      }
    }

    stage('Deploy to ECS') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: 'aws_thegk_cred',
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          sh '''
            aws ecs update-service \
              --cluster static-web-app-cluster \
              --service static-web-app-service \
              --force-new-deployment \
              --region ${AWS_REGION}
          '''
        }
      }
    }
  }
}
