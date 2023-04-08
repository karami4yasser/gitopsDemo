pipeline {
    agent any
    stages {
        stage('Print Branch Name') {
            steps {
                sh 'echo "Building on branch ${env.GIT_BRANCH}"'
            }
        }
        
        stage('ArgoCD Create') {
            steps {
                withCredentials([string(credentialsId: 'jenkins-deploy-role', variable: 'JENKINS_TOKEN')]) {
                    sh 'argocd login --username=admin --password=${JENKINS_TOKEN} '
                    sh 'argocd appset create applicationSetTest.yaml --path ${env.GIT_BRANCH}"  --dest-namespace ${env.GIT_BRANCH}"  --upsert'
                }
            }
        }
    }
}
