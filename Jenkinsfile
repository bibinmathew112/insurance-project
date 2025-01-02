node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName

    stage('Prepare Environment') {
        echo 'Initializing all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out the code from the Git repository'
            git 'https://github.com/bibinmathew112/insurance-project.git'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext(
                body: """
                    Dear All,
                    The Jenkins job ${JOB_NAME} has failed. Please review it at:
                    ${BUILD_URL}
                """,
                subject: "Job ${JOB_NAME} ${BUILD_NUMBER} failed",
                to: 'bibinmath009@gmail.com'
            )
            error('Stopping pipeline due to failure in Git checkout')
        }
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report'
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t bibinlincy/insure-me:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub'
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh "${dockerCMD} login -u bibinlincy -p ${dockerHubPassword}"
            sh "${dockerCMD} push bibinlincy/insure-me:${tagName}"
        }
    }

    stage('Configure and Deploy to Test Server') {
        ansiblePlaybook(
            become: true,
            credentialsId: 'ansible-key',
            disableHostKeyChecking: true,
            installation: 'ansible',
            inventory: '/etc/ansible/hosts',
            playbook: 'ansible-playbook.yml'
        )
    }
}
