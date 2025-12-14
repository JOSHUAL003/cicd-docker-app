pipeline {
	agent any
parameters {
	choice(
	 name: 'ENV',
	 choices: ['dev', 'prod', 'qa'],
	 description: 'Select deployment environment'
	)

	 string(
	  name: 'IMAGE_TAG',
	  defaultValue:'v1',
	  description: 'Docker Image tag'
      
         )

          boolean(
           name: 'DEPLOY',
	   defaultValue: true,
	   description: Deploy Application or not'

	  )
}

	environment {
		IMAGE_NAME= "joshualp/flask-app"
		CONTAINER_NAME= "cicd-app-${ENV}
		APP_PORT= "5000'
		EMAIL_TO= "joshualpoulose@gmail.com"
	}

	stages {
	   stage('Checkout Code') {
		steps {
			git 'https://github.com/JOSHUAL003/cicd-flask-app'
	}
}
	    stage('Install and Testing in Parallel') {
		 parallel {
		 stage('Install Dependencies') {
			 steps {
				sh 'pip install requirements.txt'
			}
		}
	          stage('Run Tests'){
			steps {
				sh 'pytest test_app.py'
			}
		}
	}
}

	stage('Build Docker Image'){
		steps {
			sh """
				docker build -t $IMAGE_NAME:${IMAGE_TAG}
			"""
	}
}
	stage('Push Image to Docker Hub') {
		steps {
		 withCredentials([usernamePassword(
	         credentialsId: 'docker-creds'
		 usernameVariable: 'USER'
		 passwordVariable: 'PASS'
		]) {
		sh"""
			echo \$PASS | echo login -u \$USER --password-stdin
			docker push $IMAGE_NAME:${IMAGE_TAG}
		"""
		}
	}
}

	 stage ('Deploy Application') {
		when {
			expression { params.DEPLOY==true}
		}
		steps{
			sh """
			docker rm -f $CONTAINER || true
			docker run -d -p $APP_PORT --name $CONTAINER $IMAGE_NAME:$IMAGE_TAG
		      """
	}
   }
}

	post {
	
	 success {
		echo "Deployment successful to ${ENV}
		mail to: "$EMAIL_TO"
			subject: "SUCCESS: Jenkins pipeline deployed to $ENV"
			body: "Build #${BUILD_NUMBER} succeeded and deployed to $ENV"
		}
		
	failure {
                echo "Deployment failure to ${ENV}
                mail to: "$EMAIL_TO"
                        subject: "FAILURE: Jenkins pipeline failed to deploy to$ENV"
                        body: "Build #${BUILD_NUMBER} failed"
                }

        always {
           echo "Pipeline execution completed"
           archiveArtifacts artifacts: '**/*.py', fingerprint: true
    }
  }
}
