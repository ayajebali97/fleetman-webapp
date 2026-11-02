def commit_id
pipeline {
    agent any
    environment {
        IMAGE = 'TON_USER_DOCKERHUB/fleetman-webapp'
    }
    stages {
        stage('Preparation') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
        stage('Image Build') {
            steps {
                echo 'Building....'
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh "docker build -t ${env.IMAGE}:${commit_id} ./"
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh "docker push ${env.IMAGE}:${commit_id}"
                }
                echo 'build complete'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'
                sh "sed -i -r 's|richardchesterwood/k8s-fleetman-webapp-angular:release2|${env.IMAGE}:${commit_id}|' ./manifests/workloads.yaml"
                sh 'kubectl get all'
                sh 'kubectl apply -f ./manifests/'
                sh 'kubectl get all'
                echo 'deployment complete'
            }
        }
    }
}
