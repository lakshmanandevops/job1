node {

  def docker = tool name: 'dockerruntime', type: 'dockerTool'
  def docCMD = "${docker}/bin/docker"
  def mvnHome = tool name: 'Maven', type: 'maven'
  def mvnCMD = "${mvnHome}/bin/mvn"
  def imageName = "laxans16/job1"

  stage('GIT Checkout') 
  {
        git 'https://github.com/lakshmanandevops/job1.git'
    
 }
 
    stage('Compile/Test/Sonar Analysus /Package') {
        
        withSonarQubeEnv('SonarServ') 
          {
        sh "${mvnCMD} clean package sonar:sonar"
          }
         
    }

    stage('Docker Build') {
        
        sh "sudo ${docCMD} build -t ${imageName} ."
    }

    stage('Docker Push') {
        withCredentials([string(credentialsId: 'dockerPwd', variable: 'dockerHubPwd')]) {
            sh "sudo ${docCMD} login -u laxans16 -p ${dockerHubPwd}"
        }
        sh "sudo ${docCMD} push ${imageName}"
        sh "sudo ${docCMD} system prune -af"
    }

    

  stage('configure ec2 with ansible') {
    ansiblePlaybook becomeUser: 'ec2-user',
    credentialsId: 'aswcred',
    installation: 'Ansible',
    playbook: 'task.yml',
    sudoUser: 'ec2-user'
  }

  def ipAddress = "NULL"
  stage('identify IP address') {
    def command = 'sudo aws ec2 describe-instances --query "Reservations[*].Instances[*].PublicIpAddress[]"'
    def output = sh script: "${command}",
    returnStdout: true
    def myIp = output.split('"')
    ipAddress = myIp[3]
    println ipAddress

  }

  stage('Install Docker on AWS') {
    def dockerCMD = 'sudo yum install docker -y'
    sshagent(['aws-server']) {
      sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"
    }
  }

  stage('Run Docker Daemon') {
    def dockerCMD = 'sudo service docker start'
    sshagent(['aws-server']) {
       sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"
    }
  }

  stage('pull docker image') {
    def dockerCMD = "sudo docker pull ${imageName}"
    sshagent(['aws-server']) {
      sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"

    }
  }

  stage('run docker image') {
    def dockerCMD = "sudo docker run -p 8090:8090 -d ${imageName}"
    sshagent(['aws-server']) {
        sh "ssh -o StrictHostKeyChecking=no ec2-user@${ipAddress} ${dockerCMD}"

    }
  }

}
