node {
	withEnv([
		"TF_CLI_ARGS=-no-color"
	]) {
		withCredentials([
			string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
			string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY'),
			string(credentialsId: 'INFRACOST_API_KEY', variable: 'INFRACOST_API_KEY'),
			string(credentialsId: 'GH_PAT', variable: 'GH_PAT')
		]) {
			stage('Plan') {
				sh "terraform init"
				sh "terraform plan -out plan.tfplan"
				sh "terraform show -json plan.tfplan > plan.json"
				stash includes: 'output.json', name: 'tfplan'
			}
			stage('Infracost Breakdown') {
				unstash name: 'tfplan'
				sh "infracost breakdown --path plan.json"
			}
			if (env.CHANGE_ID) {
				stage('Infracost PR Comment') {
					sh "infracost comment github --path=infracost.json --repo=https://github.com/doublej472/terraform-cost-test --pull-request=${env.CHANGE_ID} --github-token=${GH_PAT} --behavior=update"
				}
			}
		}
	}
}
