pipeline {
  agent any
  environment {
    REGISTRY = "123456789012.dkr.ecr.us-east-1.amazonaws.com"
    IMAGE = "demo-java-app"
    SONAR_TOKEN = credentials('sonar-token-id')
    AWS_CREDS = credentials('aws-ecr-creds')
  }
  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Build & Test') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('MySonarQube') {
          sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t ${IMAGE}:$BUILD_NUMBER ."
      }
    }

    stage('Push to ECR') {
      steps {
        withAWS(credentials: 'aws-ecr-creds', region: 'us-east-1') {
          sh """
            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${REGISTRY}
            docker tag ${IMAGE}:$BUILD_NUMBER ${REGISTRY}/${IMAGE}:$BUILD_NUMBER
            docker push ${REGISTRY}/${IMAGE}:$BUILD_NUMBER
          """
        }
      }
    }
  }
}
