node{
    stage('git checkout'){
        git credentialsId: 'git-creds', url: 'https://github.com/shubhamkushwah123/devops-bootcamp2.git'
    }
    
    stage('code build & test'){
       // sh 'mvn clean package'
       
       def mavenHome = tool name: 'maven-3' , type: 'maven'
       def mavenCMD = "${mavenHome}/bin/mvn"
       sh "${mavenCMD} clean package"
    }
    
    stage('docker build'){
        sh 'docker build -t shubhamkushwah123/cicd-demo:3.0 .'
    }
    
    stage('docker push'){
     //  withCredentials([string(credentialsId: 'dockerPwd', variable: 'dockerHubPwd')]) {
     //   sh "docker login -u shubhamkushwah123 -p ${dockerHubPwd}"
     //   }
     //   sh 'docker push shubhamkushwah123/cicd-demo:3.0'
    }
    
    stage('configure ec2 with ansible'){
    //    ansiblePlaybook becomeUser: 'ec2-user', credentialsId: 'aws-dev-server', installation: 'ansible', playbook: 'my-aws-playbook.yml', sudoUser: 'ec2-user'
    }
    
    def ipAddress = "NULL"
    stage('identify IP address'){
        def command = 'aws ec2 describe-instances --query "Reservations[*].Instances[*].PublicIpAddress[]"'
        def output = sh script: "${command}" , returnStdout:true
        def myIp = output.split('"')
        ipAddress=myIp[1]
        println ipAddress
    }
    
    stage('Install Docker on AWS'){
        def dockerCMD = 'sudo yum install docker -y'
        sshagent(['aws-dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"
        }
    }
    
    stage('Run Docker Daemon'){
        def dockerCMD = 'sudo service docker start'
        sshagent(['aws-dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"
        }
    }
    
    stage('pull docker image'){
        def dockerCMD = 'sudo docker pull shubhamkushwah123/cicd-demo:3.0'
        sshagent(['aws-dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"
        }
    }
    
    stage('run docker image'){
        def dockerCMD = 'sudo docker run -p 80:8082 -d shubhamkushwah123/cicd-demo:3.0'
        sshagent(['aws-dev-server']) {
        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"
        }
    }
}
