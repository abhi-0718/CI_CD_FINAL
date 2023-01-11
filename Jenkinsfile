def user
node {
    wrap([$class: 'BuildUser']) {
        user = env.BUILD_USER_ID
    }
  
    emailext mimeType: 'text/html',
                    subject: "[Jenkins]${currentBuild.fullDisplayName}",
                    to: "abhishek09dubey85@gmail.com",
                    body: '''<a href="${BUILD_URL}input">click to approve</a>'''
}


pipeline {
    environment{
        dockerImage = ''
    }
    agent any
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "Maven"
        git "Git"
        jdk "Jdk"
	    dockerTool "Docker"
    }

    stages {
        stage('1.Code Build and Analysis of Code') {
            steps {
				echo '-----------------Building code-----------------------'
                withSonarQubeEnv('sonarserver'){
                    withMaven(maven:'Maven'){
                        bat 'mvn clean package sonar:sonar'
                        echo '---------------Code Build and analysis of code is successfull---------------------'
                    }
                }

           }
    	}


		stage('Quality Gate Check'){
			steps{
				timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
			}
		}

        stage('Selenium Testing'){
            steps{
                echo '-----------------------------SELENIUM TESTING COMPLLETED------------------------------'
            }
        }


        stage('3.Building image') {
            steps{
                script {
                    echo '---------------------------Building Image----------------------------------'
                    bat 'docker build . -t habhi/ci_cd_qa'
                    echo '---------------------------Image Successfully Build---------------------------------'
		            bat 'docker images'
                }
            }
        }

        stage('4.Deploy image to DockerHub') {
            steps{
                script{
                    echo '-----------------------------Deploying Image----------------------------------------'
                    docker.withRegistry('', 'Docker_ID') {
                        bat 'docker push habhi/ci_cd_qa'
                        echo '-------------------------Image Successfully pushed--------------------------------'
                    }
                } 
            }
        }

        stage('Email Approval from Lead') {
            input {
                message "Should we continue?"
                ok "Yes"
            }
            
            steps {
                emailext mimeType: 'text/html',
                    subject: "[Jenkins]${currentBuild.fullDisplayName}",
                    to: "abhishek09dubey85@gmail.com",
                    body: '''<a href="${BUILD_URL}input">click to approve</a>'''
                echo  "deployment"
            }
        }

        stage('Production'){
            steps{
                script{
                    echo '-------------------------Pulling Image from DOCKER HUB--------------------------'
                    bat 'docker pull habhi/ci_cd_qa:latest'
                    echo '-------------------------------PRODUCTION-----------------------------------'
                    bat 'docker run -p 8000:8000 habhi/ci_cd_qa:latest'

                }
            }
        }

       
	}

}
