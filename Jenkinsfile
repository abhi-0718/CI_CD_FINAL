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
                // Get some code from a GitHub repository
                //git 'https://github.com/jglick/simple-maven-project-with-tests.git'
                
                // To run Maven on a Windows agent, use
				echo '-----------------Building code-----------------------'
                
                // bat "mvn clean package"
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

        stage('3.Building image') {
            steps{
                script {
                    echo '---------------------------Building Image----------------------------------'
                    bat 'docker build . -t habhi/ci_cd_dev'
                    echo '---------------------------Image Successfully Build---------------------------------'
// 		            bat 'docker images'
                }
            }
        }

         stage('4.Deploy image to DockerHub') {
            steps{
                script{
                    echo '-----------------------------Deploying Image----------------------------------------'
                    docker.withRegistry('', 'Docker_ID') {
                        bat 'docker push habhi/ci_cd_dev'
                        echo '-------------------------Image Successfully pushed--------------------------------'
                    }
                } 
            }
        }
	  
	 stage('5.Email-DEV'){
            steps{
                script{
                    emailext body: 'Deploying Project to production', subject: 'DEPLOYMENT', to: 'abhishek09dubey85@gmail.com'
                }
            }
        }
	    
	 
	    stage('6.Production'){
		    steps{
			    script{
				    echo '------------------------------PRDDUCTION------------------------------'
				    bat ' docker pull habhi/ci_cd_dev'
				    bat 'docker run -p 8000:8000 habhi/ci_cd_dev'
			    }
		    }
	    }

       
	}

}
