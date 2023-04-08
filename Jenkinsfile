pipeline {
    agent any
    stages {
        stage('Print Branch Name') {
            steps {
                echo 'Pulling...' + env.BRANCH_NAME
            }
        }
        stage('Update YAML File') {
            steps {
                script {
                    def yaml = readYaml file: 'applicationSetTest.yaml'
                    yaml.metadata.name = env.GIT_BRANCH + '-testapp'
                    yaml.spec.template.metadata.name = '{{path}}-testapp' + env.GIT_BRANCH 
                    yaml.spec.template.spec.source.targetRevision = env.GIT_BRANCH
                    yaml.spec.template.spec.destination.namespace = env.GIT_BRANCH
                    writeYaml file: 'applicationSetTesti.yaml', data: yaml
                }
            }
        }
        stage('Print yaml file') {
            steps {
                sh 'cat applicationSetTesti.yaml'
            }
        }
        
        stage('ArgoCD Create') {
            steps {
                withCredentials([string(credentialsId: 'jenkins-deploy-role', variable: 'JENKINS_TOKEN')]) {
                    sh 'argocd login localhost:8080 --username=admin --password=Pieldetoro1@ --insecure '
                    sh 'argocd appset create applicationSetTesti.yaml  --upsert'
                }
            }
        }
    }
}
