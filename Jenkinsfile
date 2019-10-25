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
                   sudo yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64 -y
                   sudo yum install -y maven
                '''
            }
        }
        stage('pre_build') {
            steps {
                sh 'echo This is pre build stage'
                sh '''
                   TAG="$REPOSITORY_NAME.$REPOSITORY_BRANCH.$ENVIRONMENT_NAME.$(date +%Y-%m-%d.%H.%M.%S).$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | head -c 8)"
                   sed -i 's@CONTAINER_IMAGE@'"$REPOSITORY_URI:$TAG"'@' app_deploy_consolidate.yml
                   $(aws ecr get-login --no-include-email --region us-east-1)
                   export KUBECONFIG=$HOME/.kube/config
                '''
            }
        }
        stage('build') {
            steps {
                sh 'echo "This is build stage"'
                sh '''
                   echo Build started on `date`
                   mvn clean package
                   #docker image build --tag $REPOSITORY_URI .
                '''
            }
        }

        stage('Docker_build'){
           docker.build('tenantthreerepo')
        }

        stage('Docker_push'){
        docker.withRegistry('https://682651395775.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:jenkins_ecr_id') {
          docker.image('tenantthreerepo').push('latest')

            }
        }

        stage('post_build') {
            steps {
                sh 'echo "This is post build stage"'
                sh '''ecr
                   echo Build completed on `date`i
                   docker push $REPOSITORY_URI:$TAG
                   CREDENTIALS=$(aws sts assume-role --role-arn arn:aws:iam::682651395775:role/tenantthreenamespace_role --role-session-name codebuild-kubectl --duration-seconds 900)
                   export AWS_ACCESS_KEY_ID="$(echo ${CREDENTIALS} | jq -r '.Credentials.AccessKeyId')"
                   export AWS_SECRET_ACCESS_KEY="$(echo ${CREDENTIALS} | jq -r '.Credentials.SecretAccessKey')"
                   export AWS_SESSION_TOKEN="$(echo ${CREDENTIALS} | jq -r '.Credentials.SessionToken')"
                   export AWS_EXPIRATION=$(echo ${CREDENTIALS} | jq -r '.Credentials.Expiration')
                   aws eks update-kubeconfig --name $EKS_CLUSTER_NAME
                   kubectl apply -f app_deploy_consolidate.yml -n tenantonenamespace
                   printf '[{"name":"appdeploy","imageUri":"%s"}]' $REPOSITORY_URI:$TAG > build.json
                '''
            }
        }
    }
