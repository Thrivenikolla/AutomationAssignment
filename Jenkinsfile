pipeline {
    agent any
	environment {
	UUID uuid = UUID.randomUUID()
        version = uuid.toString()
        registryCredential ='docker'
	containerName = "shraddhal/seleniumtest2"
        container_version = "1.0.0.${BUILD_ID}"
        dockerTag = "${containerName}:${container_version}"
    }
	tools {
        maven 'maven' 
    }
    stages {
	    
        stage('GIT clone repo and creation of version.html') {
            steps {
               //clone repo
               git 'https://github.com/vishvaja0630/AutomationAssignment.git'
			  
	       //Creating version.html and writing randomUUID to it
	       sh script:'''
	       touch musicstore/src/main/webapp/version.html
	       '''
	       println version
	       writeFile file: "musicstore/src/main/webapp/version.html", text: version
            }
        }
	    
	stage('Build maven project'){
		//cd to pom.xml
		steps{
		   sh script:'''
		   cd musicstore
		   mvn -Dmaven.test.failure.ignore=true clean package
		   '''
		  }
	}
	    
	stage('Docker build and publish tomcat image'){
		//build tomcat image with name dockerisedtomcat using Dockerfile and publish on dockerhub/shivani221	
		steps{
		    script{
			 dockerImage = docker.build("shivani221/dockerisedtomcat")
			 docker.withRegistry( '', registryCredential ) {
                         dockerImage.push("$BUILD_NUMBER")
                         dockerImage.push('latest')
			 }
			}
		}
	}    
	    
	stage('Running the tomcat container') {
		//running tomcat container with name dockerisedtomcat using the published image, port is 9090
		steps{
	         sh 'docker run -d --name dockerisedtomcat -p 9090:8080 shivani221/dockerisedtomcat:latest'
	        }
	 }
	    
	stage('Compose up for selenium test') {
		//building selenium grid for testing
                steps {
                script {
			sh 'docker-compose up -d --scale chrome=3'	
                }
	    }
        }
	    
	stage('Testing on dockerised tomcat'){
		 //testing on dockerised tomcat using selenium
		 steps{
		      sh script:'''
		      cd seleniumtest
		      mvn -Dtest="SearchTest.java" test
		      '''
		      }
		 //mvn -Dtest="UUIDTest.java" test -Duuid="$uuid"
	}  
	    
	stage('Deploy on tomcat in VM'){   
		 //deploying on VM (eg Production Environment)
                  steps{
                       deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://devopsteamgoa.westindia.cloudapp.azure.com:8081/')], contextPath: 'musicstore', onFailure: false, war: 'musicstore/target/*.war'
		       sh 'curl -sL --connect-timeout 20 --max-time 30 -w "%{http_code}\\\\n" "http://devopsteamgoa.westindia.cloudapp.azure.com:8081/musicstore/index.html" -o /dev/null'
		       script{
                       def response = sh(script: 'curl http://devopsteamgoa.westindia.cloudapp.azure.com:8081/musicstore/version.html', returnStdout: true)
		       if(env.version == response)
		       echo 'Latest version deployed'
		       else
		       echo 'Older version deployed'
	               }
	              }
                     }
        }
	
	//always running docker rm to remove tomcat image
	post{
           always{
                sh "docker rm -f dockerisedtomcat"
                 }
             }
	    
}
