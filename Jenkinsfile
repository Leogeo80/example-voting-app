pipeline {

    agent none
    
      stages {
       stage('worker build'){
           when{
                changeset "**/worker/**"
           }
           agent{
            docker {
            image 'maven:3.6.1-jdk-8-alpine'
	    args '-v $HOME/.m2:/root/.m2'
                }
            }
           steps{
             echo 'Compiling worker app'
             dir ('worker'){
               sh 'mvn compile'
             }
           }
       }
       stage('worker test'){
           when{
                changeset "**/worker/**"
           }
           agent{
            docker {
            image 'maven:3.6.1-jdk-8-alpine'
	    args '-v $HOME/.m2:/root/.m2'
                }
            }
           steps{
             echo 'Running Unit Test on result app'
             dir('worker'){
               sh 'mvn clean test'
              }
           }
       }
       stage('worker package'){
            agent{
                  docker{
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                  }
                  }
            when{
              branch 'master'
              changeset "**/worker/**"
            }
            steps{
                    echo 'Packaging worker app'
                    dir('worker'){
                      sh 'mvn package -DskipTest'
                      archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                    }
                  }
       }     
       stage('worker deploy to dev'){          
           agent any          
           when{            
	        branch 'master'          
	       }          
	    steps{            
	      echo 'Deploy instavote app with docker compose'            
	      sh 'docker-compose up -d'          
	    }     
       }
       stage('worker-docker-package'){
            agent any
            when{
                changeset "**/worker/**"
		
          }
            steps{
                    echo 'Packaging worker app with docker'
                    script {
                      docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
			def workerImage =  docker.build("leogeo80/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("latest")
                      }
                    }
                  }
        }
      
    
	   stage('vote build'){
	    agent{
           	docker {
            	image 'python:2.7.16-slim'
		        args '--user root'
              }
            }
           when{
                changeset "**/vote/**"
           }   
           steps{
             echo 'Compiling vote app'
             dir ('vote'){
               sh 'pip install -r requirements.txt'
             }
           }
        }		
      	stage('vote test'){
	    agent{
                docker {
            	image 'python:2.7.16-slim'
		args '--user root'
                }
            }

           when{
                changeset "**/vote/**"
           }
           steps{
             echo 'Running Unit Test on vote app'
             dir('vote'){
                sh 'pip install -r requirements.txt'
		sh 'nosetests -v'
	      }
           }
       }
       stage('vote docker-package'){
          agent any
             when{
            	changeset "**/vote/**"
           
          }
	     steps{
              	echo 'Packaging vote app with docker'
              	script{
                  	docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
		  	def workerImage = docker.build("leogeo80/vote:v${env.BUILD_ID}", "./vote")
                  	workerImage.push()
                  	workerImage.push("latest")
                       }
                    }
                  }
        }
   
    
       stage('result build'){
           when{
                changeset "**/result/**"
           }
           agent{
            docker {
            image 'node:8.16.0-alpine'
                }
            }
           steps{
             echo 'Compiling result app'
             dir ('result'){
               sh 'npm install'
             }
           }
       }
       stage('result test'){
           when{
                changeset "**/result/**"
           }
           agent{
            docker {
            image 'node:8.16.0-alpine'
                }
            }
           steps{
             echo 'Running Unit Test on result app'
             dir('result'){
                sh 'npm install'
                sh 'npm test'
              }
           }
       }
       stage('result docker-package'){
            agent any
            when{
                changeset "**/result/**"
		
          }
            steps{
                    echo 'Packaging result app with docker'
                    script {
                      docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin'){
			def workerImage =  docker.build("leogeo80/result:v${env.BUILD_ID}", "./result")
                        workerImage.push()
                        workerImage.push("latest")
                      }
                    }
                  }
        }
    }
      post{
	    always{
	      echo 'Pipeline for instavote app is complete .....'
	    }
      }
}
