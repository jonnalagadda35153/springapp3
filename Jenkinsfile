pipeline {
    agent any
    stages {
        stage('Install') {
            steps {
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
           }
        }
        stage('Pre_Build') {
            steps {
               sh 'TAG="$REPOSITORY_NAME.$REPOSITORY_BRANCH.$ENVIRONMENT_NAME.$(date +%Y-%m-%d.%H.%M.%S).$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | head -c 8)"'
               sh 'sed -i 's@CONTAINER_IMAGE@'"$REPOSITORY_URI:$TAG"'@' app_deploy_consolidate.yml'
               sh '$(aws ecr get-login --no-include-email)'
               sh 'export KUBECONFIG=$HOME/.kube/config'
            }
        }
        stage('Pre_Build') {
             steps {
                sh 'echo Entering build phase...'
                sh 'echo Build started on `date`'
                sh 'mvn clean package'
                sh 'docker image build --tag $REPOSITORY_URI:$TAG .'

              }
         }
        stage('Build') {
             steps {
              sh 'echo Entering post_build phase...'
              sh 'echo Build completed on `date`i'
              sh 'docker push $REPOSITORY_URI:$TAG'
              sh 'CREDENTIALS=$(aws sts assume-role --role-arn arn:aws:iam::682651395775:role/tenantthreenamespace_role --role-session-name codebuild-kubectl --duration-seconds 900)'
              sh 'export AWS_ACCESS_KEY_ID="$(echo ${CREDENTIALS} | jq -r '.Credentials.AccessKeyId')"'
              sh 'export AWS_SECRET_ACCESS_KEY="$(echo ${CREDENTIALS} | jq -r '.Credentials.SecretAccessKey')"'
              sh 'export AWS_SESSION_TOKEN="$(echo ${CREDENTIALS} | jq -r '.Credentials.SessionToken')"'
              sh 'export AWS_EXPIRATION=$(echo ${CREDENTIALS} | jq -r '.Credentials.Expiration')'
              sh 'aws eks update-kubeconfig --name $EKS_CLUSTER_NAME'
              sh 'kubectl apply -f app_deploy_consolidate.yml -n tenantonenamespace'
              sh 'printf '[{"name":"appdeploy","imageUri":"%s"}]' $REPOSITORY_URI:$TAG > build.json'

              }
         }
        post {
                always {
                    archiveArtifacts artifacts: 'target/gs-spring-boot-docker-0.1.0.jar', fingerprint: true
                }
            }
    }

}