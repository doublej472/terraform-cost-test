pipeline {
	agent any

	environment {
        	AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        	AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
		INFRACOST_API_KEY = credentials('INFRACOST_API_KEY')
		TF_CLI_ARGS="-no-color"
	}

	stages {
		stage('Plan') {
			steps {
				sh "terraform init"
				sh "terraform plan"
				sh "infracost breaakdown"
			}
		}
	}
}
