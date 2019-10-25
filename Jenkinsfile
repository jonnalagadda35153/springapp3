pipeline {
    agent any
    stages {
        stage('install') {
            steps {
                sh 'echo "This is install stage"'
                sh '''
                   curl -sS -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/aws-iam-authenticator
                   curl -sS -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/kubectl
                   chmod +x ./kubectl ./aws-iam-authenticator
                   sudo export PATH=$PWD/:$PATH
                   sudo apt-get update && apt-get -y install jq python3-pip python3-dev && pip3 install --upgrade awscli
                   echo installing maven...
                   sudo apt-get update -y
                   sudo apt-get install software-properties-common
                   sudo add-apt-repository ppa:openjdk-r/ppa
                   sudo apt-get update
                   sudo apt-get install -y openjdk-8-jdk
                   sudo apt-get install -y maven
                '''
            }
        }
    }
}