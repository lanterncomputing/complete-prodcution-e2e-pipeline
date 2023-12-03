pipeline{
    agent{
        label "jnlp-agent-jdk11"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
	dockerTool 'Docker'
    }
    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        REGISTRY = 'https://harbor.lantern-computing.com'
	HARBOR_CREDENTIAL ='harbor'
	HARBOR_NAMESPACE = 'test'
        IMAGE_NAME = "${HARBOR_NAMESPACE}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }

        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/dmancloud/complete-prodcution-e2e-pipeline'
            }

        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }
      
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }

        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }

        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry(REGISTRY, HARBOR_CREDENTIAL) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry(REGISTRY,HARBOR_CREDENTIAL) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

        }
	    
      /*  stage("Trivy Scan") {
            steps {
                script {
		   sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image dmancloud/complete-prodcution-e2e-pipeline:1.0.0-22 --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }

        }

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }


        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'https://jenkins.dev.dman.cloud/job/gitops-complete-pipeline/buildWithParameters?token=gitops-token'"
                }
            }

        }*/

    }

   /* post {
        failure {
            emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Failed", 
                    mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
            }
         success {
               emailext body: '''${SCRIPT, template="groovy-html.template"}''', 
                    subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Successful", 
                    mimeType: 'text/html',to: "dmistry@yourhostdirect.com"
          }      
    }*/
}
