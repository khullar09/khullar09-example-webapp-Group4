pipeline {
  agent any
  environment {
    AWS_DEFAULT_REGION  = 'us-east-2'
    ECR_REPOSITORY      = '990317434361.dkr.ecr.us-east-2.amazonaws.com/hello-world'
    ECS_CLUSTER_NAME    = 'webapp-cluster2'
    ECS_SERVICE_NAME    = 'webapp-cluster2-servicetask'
  }
  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main',
            url:    'https://github.com/khullar09/khullar09-example-webapp-Group4.git'
      }
    }
    stage('Verify Dockerfile') {
      steps {
        sh 'ls -la'
        sh 'test -f Dockerfile || (echo "Dockerfile not found!" && exit 1)'
      }
    }
    stage('Check AWS Credentials') {
      steps {
        withCredentials([[
          $class:       'AmazonWebServicesCredentialsBinding',
          credentialsId:'aws-access-key-id'
        ]]) {
          sh 'aws sts get-caller-identity'
        }
      }
    }
    stage('Build & Push') {
      steps {
        sh '''
          docker build -t example-webapp .
          $(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)
          docker tag example-webapp:latest $ECR_REPOSITORY:latest
          docker push $ECR_REPOSITORY:latest
        '''
      }
    }
    // you can add back your Update ECS Service here once it's working
    stage('Update ECS Service') {
  steps {
    script {
      // grab the most recent revision of our family
      def taskDefArn = sh(
        script: """
          aws ecs list-task-definitions \\
            --family-prefix webapp-task \\
            --sort DESC \\
            --max-items 1 \\
            --query 'taskDefinitionArns[0]' \\
            --output text
        """,
        returnStdout: true
      ).trim()
      echo "Updating ECS service to use task definition ${taskDefArn}"

      // now push the update
      sh """
        aws ecs update-service \\
          --cluster ${env.ECS_CLUSTER_NAME} \\
          --service ${env.ECS_SERVICE_NAME} \\
          --task-definition ${taskDefArn}
      """
    }
  }
}

  }
  post {
    success { echo '✅ Done!' }
    failure { echo '❌ Something broke.' }
  }
}
