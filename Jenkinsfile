     // Helper function to find an available port
  def findAvailablePort() {
    def socket = new ServerSocket(0)
    def port = socket.getLocalPort()
    socket.close()
    return port
  }
pipeline {
    agent any
    stages {
        stage('Print Branch Name') {
            steps {
                echo 'Pulling...' + env.BRANCH_NAME
            }
        }
        
    stage('Check if the Kind test cluster exists') {
      steps {
        script {
          def branchName = env.GIT_BRANCH 
          def clusterName = "${branchName.hashCode()}"
          def port = findAvailablePort()
         
          // Check if cluster exists
          def cmd = "kind get clusters | grep kind-${clusterName} || true"
          def result = sh(returnStdout: true, script: cmd).trim()

          // Create cluster if it doesn't exist
          if (result != clusterName) {
            // custumization of the cluster config
            def yaml = readYaml file: 'kind-test-cluster-config.yaml'
            yaml.metadata.networking.apiServerPort = port
            writeYaml file: 'clusterConfig.yaml', data: yaml
            echo "Kind cluster kind-${clusterName} does not exist, creating..."
            sh "kind create cluster --name ${clusterName} --config clusterConfig.yaml"
             echo "Kind cluster kind-${clusterName} created"

            echo "Registring the cluster kind-${clusterName} for argocd "
            
            sh 'argocd login localhost:8080 --username=admin --password=Pieldetoro1@ --insecure '
            sh "argocd cluster add kind-${clusterName}  --yes"

          } else {
            echo "Kind cluster kind-${clusterName} already exists."
          }
        }
      }
    }
    stage('Update ApplicationSet YAML File') {
            steps {
                script {
                    def yaml = readYaml file: 'applicationSetTest.yaml'
                    yaml.metadata.name = env.GIT_BRANCH + '-testapp'
                    yaml.spec.template.metadata.name = '{{path}}-testapp' + env.GIT_BRANCH 
                    yaml.spec.template.spec.source.targetRevision = env.GIT_BRANCH
                    yaml.spec.template.spec.destination.name = 'kind-'+env.GIT_BRANCH.hashCode()
                    writeYaml file: 'applicationSetTestModified.yaml', data: yaml
                }
            }
        }
        stage('Print applicationSetTestModified.yaml file') {
            steps {
                sh 'cat applicationSetTestModified.yaml'
               
            }
        }
        
        stage('ArgoCD Create') {
            steps {
                withCredentials([string(credentialsId: 'jenkins-deploy-role', variable: 'JENKINS_TOKEN')]) {
                    sh 'argocd login localhost:8080 --username=admin --password=Pieldetoro1@ --insecure '
                    sh 'argocd appset create applicationSetTestModified.yaml  --upsert'
                }
            }
        }
        
    }



}
