pipeline {
    agent any
    stages {
        stage("SCM"){
            steps{
                git branch: 'master', 
                credentialsId: 'git_cred', 
                url: 'https://github.com/Abhi-chintu/springboot-deploy-jenkins.git'
            }
        }
        stage("build"){
            steps{
                sh "mvn clean install"
            }
        }
        stage("docker image"){
            steps{
                sh "docker build -t springboothelloeks:${BUILD_NUMBER} ."
                sh "docker images"
            }
        }
        stage("Pushing Image"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub_cred', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh "docker login -u ${username} -p ${password}" 
                sh "docker tag springboothelloeks:${BUILD_NUMBER} ${username}/springboothelloeks:${BUILD_NUMBER}"
                sh "docker push ${username}/springboothelloeks:${BUILD_NUMBER}"
                }
            }
        }
        stage("Connect to eks cluster"){
            steps{
                sh "aws eks update-kubeconfig --region ap-south-1 --name eksdemo"
            }
        }
        stage("Deploy Manifest files"){
            steps{
                dir('k8s') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_cred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh '''
                    cat deployment.yaml | sed "s/{{reponame}}/${username}/g" | sed "s/{{ImageTag}}/${BUILD_NUMBER}/g" | kubectl apply -f -
                            kubectl apply -f service.yaml
                    '''
                    }
                }    
            }
        }
    }
}
