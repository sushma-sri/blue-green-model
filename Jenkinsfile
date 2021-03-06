pipeline{
    agent any
    stages{
        stage('Lint HTML') {
            steps {
                sh 'tidy -q -e app/*.html'
            }
        }
        stage('Build Docker Image') {
           steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                    sh '''
                        docker build --no-cache -t sushmasri/capstonerepository:latest .
                    '''
                }

            }
        }
        stage('Push Docker Image') {
           steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                    sh '''
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker push sushmasri/capstonerepository:latest
                    '''
                }
            }
        }
        stage('Set Kubectl Context to Cluster') {
            steps{
                sh 'kubectl config use-context arn:aws:eks:ap-southeast-2:984760320768:cluster/capstonecluster'
            }
        }
        stage('Create Blue Controller') {
            when {
                expression { env.BRANCH_NAME == 'blue' }
            }
            steps{
                withAWS(region:'ap-southeast-2',credentials:'aws-static') {
                    sh 'kubectl apply -f ./blue-controller.json'

                }
            }
        }
        stage('Create Green Controller') {
            when {
                expression { env.BRANCH_NAME == 'green' }
            }
            steps{
                withAWS(region:'ap-southeast-2',credentials:'aws-static') {
                    sh 'kubectl apply -f ./green-controller.json'

                }
            }
        }
        
        stage('Create Blue-Green service') {
            when {
                expression { env.BRANCH_NAME != 'master' }
            }
            steps{
                withAWS(region:'ap-southeast-2',credentials:'aws-static') {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh 'kubectl apply -f ./blue-green-service.json'
                        sh 'kubectl get pods'
                        sh 'kubectl describe service bluegreenlb'
                    }
                }
            }

        }
    }
}
