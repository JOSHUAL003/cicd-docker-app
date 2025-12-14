pipeline {
  agent any

  /* =====================
     PARAMETERS
     ===================== */
  parameters {
    choice(
      name: 'ENV',
      choices: ['dev', 'qa', 'prod'],
      description: 'Select deployment environment'
    )

    string(
      name: 'IMAGE_TAG',
      defaultValue: 'v1',
      description: 'Docker image tag'
    )

    booleanParam(
      name: 'DEPLOY',
      defaultValue: true,
      description: 'Deploy application or not'
    )
  }

  /* =====================
     ENVIRONMENT VARIABLES
     ===================== */
  environment {
    IMAGE_NAME     = "joshualp/flask-app"
    CONTAINER_NAME = "cicd-app-${ENV}"
    APP_PORT       = "5000"
    EMAIL_TO       = "joshualpoulose@gmail.com"
  }

  stages {
    stage('Install and Test (Parallel)') {
      parallel {

        stage('Install Dependencies') {
          steps {
            sh 'sh 'pip install -r requirements.txt --break-system-packages'
'
          }
        }

        stage('Run Tests') {
          steps {
            sh 'pytest test_app.py'
          }
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh """
          docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
        """
      }
    }

    stage('Push Image to Docker Hub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh """
            echo \$PASS | docker login -u \$USER --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Deploy Application') {
      when {
        expression { params.DEPLOY == true }
      }
      steps {
        sh """
          docker rm -f ${CONTAINER_NAME} || true
          docker run -d -p ${APP_PORT}:5000 --name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
        """
      }
    }
  }

  /* =====================
     POST BUILD ACTIONS
     ===================== */
  post {

    success {
      echo "Deployment successful to ${ENV}"
      mail to: "${EMAIL_TO}",
           subject: "SUCCESS: Jenkins pipeline deployed to ${ENV}",
           body: "Build #${BUILD_NUMBER} succeeded and deployed to ${ENV}"
    }

    failure {
      echo "Deployment failed for ${ENV}"
      mail to: "${EMAIL_TO}",
           subject: "FAILURE: Jenkins pipeline failed for ${ENV}",
           body: "Build #${BUILD_NUMBER} failed. Check Jenkins logs."
    }

    always {
      echo "Pipeline execution completed"
      archiveArtifacts artifacts: '**/*.py', fingerprint: true
    }
  }
}
