pipeline{
    agent any
    stages{
        stage('git checkout'){
            steps{
            echo 'checkout done'
            git 'https://github.com/bibinmathew112/insurance-project.git'
            }
        }
        stage('packaging the code'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('build the docker image'){
            steps{
                sh 'docker build -t bibinlincy/insuranceproject:1 .'
            }
        }
        stage('docker hub login'){
            steps{
                withCredentials([string(credentialsId: 'dockerhubpass', variable: 'dockerhubpass')]) {
    // some block
                }
                
            }
        }
        stage('push the docker image'){
            steps{
                sh 'docker push bibinlincy/insuranceproject:1'
            }
        }
        stage('Deploy') {
            steps {
                sh 'sudo docker run -itd --name insurance-project -p 8085:8081 bibinlincy/insuranceproject:1'
            }
            
        }
    }
}
