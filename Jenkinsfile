pipeline {
    agent any
    stages {
        stage('Install') {
            steps ([
                sh 'curl -sS -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/aws-iam-authenticator'
                sh 'curl -sS -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/kubectl'
                sh 'chmod +x ./kubectl ./aws-iam-authenticator'
                sh 'export PATH=$PWD/:$PATH'
                sh 'apt-get update && apt-get -y install jq python3-pip python3-dev && pip3 install --upgrade awscli'
                sh 'echo installing maven...'
                sh 'apt-get update -y'
                sh 'apt-get install software-properties-common'
                sh 'add-apt-repository ppa:openjdk-r/ppa'
                sh 'apt-get update'
                sh 'apt-get install -y openjdk-8-jdk'
                sh 'apt-get install -y maven'
           ])
        }

    }
}