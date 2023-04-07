pipeline {
agent any 
tools {
maven 'maven'
}

stages {
stage("Git Checkout"){
steps{
git 'https://github.com/sandeepajay1/medicure.git'
 }
 }
stage('Build the application'){
steps{
echo 'cleaning..compiling..testing..packaging..'
sh 'mvn clean install package'
 }
 }
 
stage('Publish HTML Report'){
steps{
    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/medicure/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
 }
}
stage('Docker build image') {
              steps {
                  
                  sh'sudo docker system prune -af '
                  sh 'sudo docker build -t sandeepajay1/medicure:latest . '
              
                }
            }
stage('Docker login and push') {
              steps {
                   withCredentials([string(credentialsId: 'docpass', variable: 'docpasswd')]) {
                  sh 'sudo docker login -u sandeepajay1 -p ${docpasswd} '
                  sh 'sudo docker push sandeepajay1/medicure:latest'
                  }
                }
        }    
 stage (' setting up Kubernetes with terraform '){
            steps{

                dir('terraform_files'){
                sh 'terraform init'
                sh 'terraform validate'
                sh 'terraform apply --auto-approve'
                sh 'sleep 20'
                }
               
            }
        }
stage('deploy application to kubernetes'){
steps{
sh 'sudo chmod 600 ./terraform_files/DEMOKEY.pem'    
sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/DEMOKEY.pem medicure-deployment.yml ubuntu@54.87.119.199:/home/ubuntu/'
sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/DEMOKEY.pem medicure-service.yml ubuntu@54.87.119.199:/home/ubuntu/'
script{
try{
sh 'ssh -i -o StrictHostKeyChecking=no ./terraform_files/DEMOKEY.pem ubuntu@54.87.119.199 kubectl apply -f .'
}catch(error)
{
sh 'ssh -i -o StrictHostKeyChecking=no ./terraform_files/DEMOKEY.pem ubuntu@54.87.119.199 kubectl apply -f .'
}
}
}
}
}
}
