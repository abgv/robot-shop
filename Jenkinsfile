podTemplate (label: 'robotshop', containers: [
		containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
		containerTemplate(name: 'kubectl', image: 'boxboat/kubectl:latest', command: 'cat', ttyEnabled: true)
	],
	volumes: [
		hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
	]) {
	node ('robotshop') {
		def repo = checkout scm
		def gitCommit = repo.GIT_COMMIT
    	def gitBranch = repo.GIT_BRANCH
		def shortGitCommit = "${gitCommit[0..10]}"
		def changes = sh (script: "git diff --dirstat=files,0 HEAD~1 | sed -E 's/^[ 0-9.]+% //g' | sed -E 's/\\/.*\$//g'", returnStdout: true)
		changes = changes.split('\\n')
		
		stage ('Build') {
			container ('docker') {
				for (change in changes) {
					change.trim()
					sh "echo ${change}"
					def buildable = !(change.matches(".*\\b(K8s|DCOS|OpenShift|Swarm)\\b.*"))
					if (change != '' && buildable) {
						dir (change) {
							if (change == 'mysql') {
									change = "${change}-db"
							}
							else if (change == 'mongo') {
								change = "${change}db"
							}

							withDockerRegistry(credentialsId: 'DockerHub') {
								def tag = "${env.DOCKER_REGISTRY}/rs-${change}:latest"
								sh """
								docker build -t ${tag} .
								docker push ${tag}
								"""
							}
						}
					}		
					else
					{
						sh 'echo No buildable Microservice'
					}
				}	
			}
		}

		stage ('Deploy') {
			container ('kubectl') {
				for (change in changes) {
					change.trim()
					def buildable = !(change.matches(".*\\b(K8s|DCOS|OpenShift|Swarm)\\b.*"))
					if (change != '' && buildable) {
						if (change == 'mongo') {
							change = "${change}db"
						}

						sh """
						kubeclt apply -f ./K8s/descriptors/${change}-deployment.yaml
						kubeclt apply -f ./K8s/descriptors/${change}-service.yaml
						kubectl set image deployment/${change} ${change}=${env.DOCKER_REGISTRY}/rs-${change}:latest -n robot-shop --record
						"""
					}
				}
			}
		}
	}
}
