podTemplate (label: 'robotshop_pod', containers: [
		containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
	],
	volumes: [
		hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
	]) {
	node ('robotshop_pod') {
		def repo = checkout scm
		def gitCommit = repo.GIT_COMMIT
    	def gitBranch = repo.GIT_BRANCH
		def changes = sh(script: "git diff --dirstat=files,0 HEAD~1 | sed -E 's/^[ 0-9.]+% //g' | sed -E 's/\\/.*$//g'", returnStdout: true)

		stage ('Build') {
			container('docker') {
				sh """
				pwd
				echo "GIT_BRANCH=${gitBranch}" 
				echo "GIT_COMMIT=${gitCommit}" 
				echo "CHANGES=${changes}"
				"""
			}
		}
	}
}
