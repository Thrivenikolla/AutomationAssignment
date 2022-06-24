pipeline {
    agent any
	environment {
        UUID uuid = UUID.randomUUID()
        registryCredential ='docker_cred'
	containerName = "shraddhal/seleniumtest2"
        container_version = "1.0.0.${BUILD_ID}"
        dockerTag = "${containerName}:${container_version}"
        }
	
	tools {
        maven '3.8.5' 
        }
	
    stages {
	    stage('GIT clone repo and creation of version.html') {
            steps {
               //clone repo
               git 'https://github.com/Thrivenikolla/AutomationAssignment.git'
			  
	       //Creating version.html and writing randomUUID to it
	       sh script:'''
	       touch musicstore/src/main/webapp/version.html
	       '''
	       println uuid
	       writeFile file: "musicstore/src/main/webapp/version.html", text: uuid
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
			 dockerImage = docker.build("thrivenik/dockerisedtomcat")
			 docker.withRegistry('https://hub.docker.com', registryCredential) {
                         dockerImage.push("$BUILD_NUMBER")
                         dockerImage.push('latest')
			 }
			 }
		    }
	  }    
	    
	stage('Running the tomcat container') {
		//running tomcat container with name dockerisedtomcat using the published image, port is 9090
		steps{
	         sh 'docker run -d --name dockerisedtomcat -p 9090:8080 thrivenik/dockerisedtomcat:latest'
	        }
	 }
	    
	/*  stage('Compose up for selenium test') {
		//building selenium grid for testing
                steps {
                script {
			sh 'docker-compose up -d --scale chrome=3'	
                }
	        }
        } 
	    
	stage('Testing on dockerised tomcat'){
		 //testing on dockerised tomcat using selenium(3 tests: UUID, SearchTest for string matching and SearchTest2 for failure)
		 steps{
		      sh script:'''
		      cd seleniumtest
		      mvn -Dtest="SearchTest.java" test
		      '''
		      }
		 //mvn -Dtest="UUIDTest.java" test -Duuid="$uuid"
		 //mvn -Dtest="SearchTest2.java" test
	}  */
	    
	stage('Deploy on tomcat in VM'){   
		 //deploying on VM (eg Production Environment)
                  steps{
                       deploy adapters: [tomcat9(credentialsId: 'bd198682-9ac4-4fc7-8669-922e90d3093b', path: '', url: 'https://hub.docker.com/repository/create?namespace=thrivenik')], contextPath: 'musicstore', onFailure: false, war: 'musicstore/target/*.war'
		       sh 'curl -sL --connect-timeout 20 --max-time 30 -w "%{http_code}\\\\n" "https://hub.docker.com/repository/create?namespace=thrivenik/musicstore/index.html" -o /dev/null'
		       script{
                       def response = sh(script: 'curl https://hub.docker.com/repository/create?namespace=thrivenik/musicstore/version.html', returnStdout: true)
		       if(env.uuid == response)
		       echo 'Latest version deployed'
		       else
		       echo 'Older version deployed'
	               }
	               }
        }
        }//stages closed
	
	//always running docker rm to remove dockerisedtomcat image and docker-compose down for selenium
	post{
           always{
                sh "docker rm -f dockerisedtomcat"
		//sh 'docker-compose down'	
                 }
             }  
}
