pipeline {
    agent any
    stages {
        stage('install') {
            steps {
                sh 'echo "This is install stage"'
                sh '''
                   curl -sS -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/aws-iam-authenticator
                   curl -sS -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/linux/amd64/kubectl
                   sudo chmod +x ./kubectl ./aws-iam-authenticator
                   export PATH=$PWD/:$PATH
                   sudo apt-get update && apt-get -y install jq python3-pip python3-dev && pip3 install --upgrade awscli
                   echo installing maven...
                   sudo yum update -y
                   #sudo yum install software-properties-common -y
                   #sudo yum install java-1.8.0-openjdk
                   #sudo add-apt-repository ppa:openjdk-r/ppa
                   sudo yum update -y
                   sudo yum install -y openjdk-8-jdk
                   sudo yum install -y maven
                '''
            }
        }
        stage('pre_build') {
            steps {
                sh 'echo "This is pre build stage"'
                sh '''
                   TAG="$REPOSITORY_NAME.$REPOSITORY_BRANCH.$ENVIRONMENT_NAME.$(date +%Y-%m-%d.%H.%M.%S).$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | head -c 8)"
                   sed -i 's@CONTAINER_IMAGE@'"$REPOSITORY_URI:$TAG"'@' app_deploy_consolidate.yml
                   $(aws ecr get-login --no-include-email)
                   export KUBECONFIG=$HOME/.kube/config
                '''
            }
        }
    }
}