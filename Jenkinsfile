pipeline{
agent {
    kubernetes {
      defaultContainer 'python'
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: python
            image: dtr-prod-virtual.artifactory.onintranet.com/python:3.8.12
            command:
            - cat
            tty: true
        '''
    }
  }
    stages{
        stage('Create image and push to Artifactory') {
            steps {
			script{
				checkout([$class: 'GitSCM', 
						branches: [ [name: '*/${GIT_BRANCH}'] ], 
						doGenerateSubmoduleConfigurations: false,
						poll: true,
						extensions: [[$class: 'CleanCheckout']], 
						submoduleCfg: [], 
						userRemoteConfigs: [[url: 'git@github.com:AtheneUSA/continuous-integration.git']]
					])			
                
				sh 'pip install pyyaml Jinja2'
				step([$class: 'StashNotifier'])         // Notifies the Stash Instance of an INPROGRESS build

				def pillarErrors = sh script:'python ./tester' ,  returnStatus:true
				
				if (pillarErrors) {
					currentBuild.result = 'FAILED'                
				}  else {
					currentBuild.result = 'SUCCESS'
				}     

				step([$class: 'StashNotifier'])
				
				if (pillarErrors) {
					sh script: 'ls -lisa'
					sh script: 'if [ -f tester_out.txt ]; then cat tester_out.txt; else echo "No output"; fi'
					error('Pillars Error')
				}
            }
			}
        }
        
	}
	post {
        success {
            script{
                currentBuild.result = 'SUCCESS'
                step([$class: 'StashNotifier'])
            }
        }
        failure {
            script{
                currentBuild.result = 'FAILED'
                step([$class: 'StashNotifier'])
            }
        }
    }
}