node {
	checkout scmGit(extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'repo']], userRemoteConfigs: scm.userRemoteConfigs)

	if (env.CHANGE_ID) {
		checkout scmGit(branches: [[name: 'refs/heads/master']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'base-repo']], userRemoteConfigs: [[credentialsId: scm.userRemoteConfigs[0].credentialsId, refspec: ' +refs/heads/master:refs/remotes/origin/master', url: scm.userRemoteConfigs[0].url]])
	}
    
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
			    sh "cd $WORKSPACE/repo"
				sh "terraform init"
				sh "terraform plan -out plan.tfplan"
				sh "terraform show -json plan.tfplan > $WORKSPACE/plan.json"
				
			    if (env.CHANGE_ID) {
			        sh "cd $WORKSPACE/base-repo"
    	            		sh "terraform init"
    				sh "terraform plan -out plan-base.tfplan"
    				sh "terraform show -json plan-base.tfplan > $WORKSPACE/plan-base.json"
			    }

			    stash includes: 'plan.json, plan-base.json', name: 'tfplan'
			}
			stage('Infracost Breakdown') {
				unstash name: 'tfplan'
				
				sh "infracost breakdown --project-name Terraform --usage-file infracost-usage.yml --path plan-base.json --format json --out-file infracost-base.json"
				
				if (env.CHANGE_ID) {
				    sh "infracost diff --compare-to infracost-base.json --project-name Terraform --usage-file infracost-usage.yml --path plan.json --format json --out-file infracost.json"
				} else {
				    sh "cp -vf infracost-base.json infracost.json"
				}
				
				sh "infracost output --path infracost.json --format html --out-file infracost.html"
				sh "infracost output --path infracost.json --format table"
				
				stash includes: 'infracost.json', name: 'infracost'
				archiveArtifacts artifacts: 'infracost.json, infracost-base.json, infracost.html, infracost-usage.yml', fingerprint: true
			}
			if (env.CHANGE_ID) {
				stage('Infracost PR Comment') {
					unstash name: 'infracost'
					sh 'infracost comment github --path=infracost.json --repo=doublej472/terraform-cost-test --pull-request=${CHANGE_ID} --github-token=${GH_PAT} --behavior=update'
				}
			}
		}
	}
}
