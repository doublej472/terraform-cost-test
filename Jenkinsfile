node {
	environment {
        	AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        	AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
		INFRACOST_API_KEY = credentials('INFRACOST_API_KEY')
		GH_PAT = credentials('GH_PAT')
		TF_CLI_ARGS="-no-color"
	}
	stage('Plan') {
		steps {
			sh "terraform init"
			sh "terraform plan -out plan.tfplan"
			sh "terraform show -json plan.tfplan > plan.json"
			stash includes: 'output.json', name: 'tfplan'
		}
	}
	stage('Infracost Breakdown') {
		steps {
			unstash name: 'tfplan'
			sh "infracost breakdown --path plan.json"
		}
	}
	if (env.CHANGE_ID) {
		stage('Infracost PR Comment') {
			steps {
				sh "infracost comment github --path=infracost.json --repo=https://github.com/doublej472/terraform-cost-test --pull-request=${env.CHANGE_ID} --github-token=${GH_PAT} --behavior=update"
			}
		}
	}
}
