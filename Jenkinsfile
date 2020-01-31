podTemplate (label: 'robotshop', containers: [
		containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
	],
	volumes: [
		hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
	]) {
	node ('robotshop') {
		def repo = checkout scm
		def gitCommit = repo.GIT_COMMIT
    	def gitBranch = repo.GIT_BRANCH
		def changes = sh(script: "git diff --dirstat=files,0 HEAD~1 | sed -E 's/^[ 0-9.]+% //g' | sed -E 's/\\/.*\$//g'", returnStdout: true)

		stage ('Build') {
			container ('docker') {
				def buildable = !(changes.matches(".*\\b(K8s|DCOS|OpenShift|Swarm)\\b.*"))
				if (changes != '' && buildable) {
					sh """
					pwd
					echo "GIT_COMMIT=${gitCommit}" 
					echo "GIT_BRANCH=${gitBranch}" 
					echo "CHANGES=${changes}"
					ls
					"""
					dir (changes) {
						withCredentials ([usernamePassword (credentialsId: 'DockerHub', 
									passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USER')]) {
							def tag = "${DOCKERHUB_USER}/rs-${changes}:${gitCommit}"
							sh """
							docker login -u ${DOCKERHUB_USER} -p '${DOCKERHUB_PASSWORD}'
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
}
