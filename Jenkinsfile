pipeline {
    agent { label 'EC2_Jenkins_Agent' } //Slaves on which you want to run your pipeline on 
    tools {    // plugins which we installed 
        jdk 'Java17'   // Name which we gave in tools for the plugin 
        maven 'Maven3'
    }
    // environment {
	   //  APP_NAME = "register-app-pipeline"
    //         RELEASE = "1.0.0"
    //         DOCKER_USER = "ashfaque9x"
    //         DOCKER_PASS = 'dockerhub'
    //         IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
    //         IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	   //  JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    // }
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
    }

    
}
