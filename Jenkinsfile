pipeline {
    agent { label 'EC2_Jenkins_Agent' } //Slaves on which you want to run your pipeline on 
    tools {    // plugins which we installed 
        jdk 'Java 17'   // Name which we gave in tools for the plugin 
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "bvr98"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    //JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'Github', url: 'https://github.com/bvr98/register-app'
			// branch       git credentials which we configured in Jenkins Credentails     URL: git repo
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package" // cleans and packages the application into binary file jar/war
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test" // runs the test cases 
           }
       }
	    stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'SonarQube-Server') { 
                        sh "mvn sonar:sonar" // Runs the code quality analysis on the project
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    //waitForQualityGate abortPipeline: true, credentialsId: 'SonarQube-Server'
		       timeout(time: 15, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
                       def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                       if (qg.status != 'OK') {
                              error "Pipeline aborted due to quality gate failure: ${qg.status}"
                     //checks for the quality gate condition, which is sent by a webhook SQ->Jenkins, if pass pipeline continues if fails abort.
                      }	
                   }
	       }
	   }
        }

	  stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }

       stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image bvr98/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
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
    }

    
}
