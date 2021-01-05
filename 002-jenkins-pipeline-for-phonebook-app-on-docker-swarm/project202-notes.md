# Project 202: Notes

## Docker Installation

```bash
yum update -y
amazon-linux-extras install docker -y
systemctl start docker
systemctl enable docker
usermod -a -G docker ec2-user
```

## Docker-Compose Installation

- https://docs.docker.com/compose/compose-file/#update_config

```bash
curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## AWS EC2 Connect CLI Installation

- http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-set-up.html

```bash
pip3 install ec2instanceconnectcli
rm -rf /bin/aws
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install
```

## AWS EC2 Credientials Helper Installation
- https://aws.amazon.com/blogs/compute/authenticating-amazon-ecr-repositories-for-docker-cli-with-credential-helper/


```bash
yum install amazon-ecr-credential-helper -y
mkdir -p /home/ec2-user/.docker
cd /home/ec2-user/.docker
echo '{"credsStore": "ecr-login"}' > config.json
```
## When we need to execute our commands and get some info from remote machine we use below command.
- With that command, we get info from remote machine, and execute it as a command our current machine.
    - in this example, we get our swarm join token from grand manager and execute this output as a command our machine

    - https://docs.docker.com/engine/swarm/swarm-tutorial/


```bash
eval "$(mssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no  \
--region ${AWS::Region} ${DockerManager1} docker swarm join-token worker | grep -i 'docker')"
```


## Swarm Vizualition Container Installation

```bash
docker service create \
--name=viz \
--publish=8080:8080/tcp \
--constraint=node.role==manager \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
dockersamples/visualizer
```

## Create a own ssh key on local machine 

- https://www.ssh.com/ssh/keygen/

```bash
ssh-keygen -t rsa
```

## Jenkinsfile (Creating ECR Repository)

```groovy
stage('creating ECR Repository') {
            steps {
                echo 'creating ECR Repository'
                sh """
                aws ecr create-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --image-scanning-configuration scanOnPush=false \
                  --image-tag-mutability MUTABLE \
                  --region ${AWS_REGION}
                """
            }
        }
```

## Jenkinsfile (Building Docker Image)

```groovy
stage('building Docker Image') {
            steps {
                echo 'building Docker Image'
                sh 'docker build --force-rm -t "$ECR_REGISTRY/$APP_REPO_NAME:latest" .'
                sh 'docker image ls'
            }
        }
```

## Jenkinsfile (Pushing Docker image to ECR)

```groovy
stage('pushing Docker image to ECR Repository'){
            steps {
                echo 'pushing Docker image to ECR Repository'
                sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh 'docker push "$ECR_REGISTRY/$APP_REPO_NAME:latest"'

            }
        }
```

## Jenkinsfile (Creating Infrastructure for the App)

```groovy
stage('creating infrastructure for the Application') {
            steps {
                echo 'creating infrastructure for the Application'
                sh "aws cloudformation create-stack --region ${AWS_REGION} --stack-name ${AWS_STACK_NAME} --capabilities CAPABILITY_IAM --template-body file://${CFN_TEMPLATE} --parameters ParameterKey=KeyPairName,ParameterValue=${CFN_KEYPAIR}"

            script {
                while(true) {
                        
                        echo "Docker Grand Master is not UP and running yet. Will try to reach again after 10 seconds..."
                        sleep(10)

                        ip = sh(script:'aws ec2 describe-instances --region ${AWS_REGION} --filters Name=tag-value,Values=docker-grand-master Name=tag-value,Values=${AWS_STACK_NAME} --query Reservations[*].Instances[*].[PublicIpAddress] --output text | sed "s/\\s*None\\s*//g"', returnStdout:true).trim()

                        if (ip.length() >= 7) {
                            echo "Docker Grand Master Public Ip Address Found: $ip"
                            env.MASTER_INSTANCE_PUBLIC_IP = "$ip"
                            break
                        }
                    }
                }
            }
        }
```

## Jenkinsfile (Deploying the Application)

```groovy
stage('Deploying the Application'){
            environment {
                MASTER_INSTANCE_ID=sh(script:'aws ec2 describe-instances --region ${AWS_REGION} --filters Name=tag-value,Values=docker-grand-master Name=tag-value,Values=${AWS_STACK_NAME} --query Reservations[*].Instances[*].[InstanceId] --output text', returnStdout:true).trim()
            }
            steps {
                echo 'Cloning and Deploying App on Swarm using Grand Master with Instance Id: $MASTER_INSTANCE_ID'
                sh 'mssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no --region ${AWS_REGION} ${MASTER_INSTANCE_ID} git clone ${GIT_URL}'
                sleep(10)
                sh 'mssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no --region ${AWS_REGION} ${MASTER_INSTANCE_ID} docker stack deploy --with-registry-auth -c ${HOME_FOLDER}/${GIT_FOLDER}/docker-compose.yml ${APP_NAME}'

            }
        }
```

## Jenkinsfile (Post Build)

```groovy
post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
        failure {
            echo 'Delete the Image Repository on ECR due to the Failure'
            sh """
                aws ecr delete-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --region ${AWS_REGION}\
                  --force
                """
            echo 'Deleting Cloudformation Stack due to the Failure'
            sh 'aws cloudformation delete-stack --region ${AWS_REGION} --stack-name ${AWS_STACK_NAME}'
        }
        success {
            echo 'You are the man/woman...'
        }
    }
```