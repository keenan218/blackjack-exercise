pipeline{
    agent any
    environment {
        app_version = 'v1'
        rollback = 'false'
    }
    stages { 
        stage('Set up') {
            steps {
                sh '''
                ssh $USER@$VM <<EOF
                if [ -d "blackjack-exercise" ]
                then
                rm -r blackjack-exercise --force
                git clone https://github.com/keenan218/blackjack-exercise.git
                else
                git clone https://github.com/keenan218/blackjack-exercise.git
                fi
                git checkout development
                ls -a
                >>EOF
                '''
            }
        }
   /*     stage('Build images') {
            steps{
                script {
                    if (env.rollback == 'false') {
                        image = docker.build("maccpr7/frontend-react", "./frontend")
                        image1 = docker.build("maccpr7/backend-java", "./backend")
                    }
                }
            }
        }
        stage ('Push images') {
            steps{
                script {
                    if (env.rollback == 'false'){
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials'){ //need to set up a dockerhub credentials on jenkins
                            image.push("${env.app_version}")
                            image1.push("${env.app_version}")
                        }
                    }                 
                }
            }
        } */
        stage('Deploy app') {
            steps { 
                sh '''
                ssh $USER@$VM <<EOF
                
                curl https://get.docker.com | sudo bash
                sudo apt update -y && sudo apt install -y curl jq
                sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                sudo chmod +x /usr/local/bin/docker-compose
                sudo usermod -aG docker ubuntu
                
                cd blackjack-exercise
                git checkout development 
                
                docker-compose up -d --build 
                >> EOF
                '''
            }
        }
        stage('Production deploy') {
            steps {              
                sh '''
                aws configure set aws_access_key_id $access_key
                aws configure set aws_secret_access_key $secret_key
                aws configure set default.region eu-west-2
                
                sudo snap install kubectl --classic       
                
                aws eks --region eu-west-2 update-kubeconfig --name project-cluster
                
                kubectl apply -f ./kubernetes
                
                sleep 60
                kubectl get pods
                kubectl get services
                sleep 30
                kubectl get pods 
                kubectl get services 
                '''
            }
        }
    }
}
