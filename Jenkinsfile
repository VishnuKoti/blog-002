#!groovy

def releasedVersion

node('master') {
  def dockerTool = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
  withEnv(["DOCKER=${dockerTool}/bin"]) {
  
    stage("Cleanup") {
    	dockerCmd "stop zalenium"
    	dockerCmd "rm zalenium"
    	dockerCmd "stop snapshot"
    	dockerCmd "rm snapshot"
    }
    
    stage('Prepare') {
        deleteDir()
        parallel Checkout: {
            checkout scm
        }, 'Run Zalenium': {
            dockerCmd '''run -d --name zalenium -p 4444:4444 \
            -v /var/run/docker.sock:/var/run/docker.sock \
            --network="host" \
            --privileged dosel/zalenium:3.4.0a start --videoRecordingEnabled false --chromeContainers 1 --firefoxContainers 0'''
        }
    }

    stage('Build') {
             git url: 'https://github.com/VishnuKoti/blog-002.git'
              dir('app') {
	        def mvnHome = tool 'M3'
 	 	sh "${mvnHome}/bin/mvn clean package"
                dockerCmd 'build --tag automatingguy/sparktodo:SNAPSHOT .'
            }
    }
    
    stage("BuildAnother"){
     		git url: 'https://github.com/VishnuKoti/spring-boot-examples.git'
                dir('spring-boot-tutorial-basics'){
                def mvnHome = tool 'M3'
    		 sh "${mvnHome}/bin/mvn clean package"
                     dockerCmd 'build --tag springguy/springguy:SNAPSHOT .'
            }
    }

    stage('Deploy') {
        stage('Deploy') {
            dir('app') {
                dockerCmd 'run -d -p 9999:9999 --name "snapshot" --network="host" automatingguy/sparktodo:SNAPSHOT'
            }
        }
    }
    
      stage('DeployAgain') {
            stage('DeployAgain') {
                 dir('spring-boot-tutorial-basics'){
                    dockerCmd 'run -d -p 9990:9990 --name "springshot" --network="host" springguy/springguy:SNAPSHOT'
                }
            }
    }
    
     stage('Tests') {
            try {
                dir('tests/rest-assured') {
                    sh './gradlew clean test'
                }
            } finally {
                junit testResults: 'tests/rest-assured/build/*.xml', allowEmptyResults: true
                archiveArtifacts 'tests/rest-assured/build/**'
            }
    
            dockerCmd 'rm -f snapshot'
            dockerCmd 'run -d -p 9999:9999 --name "snapshot" --network="host" automatingguy/sparktodo:SNAPSHOT'
    
            try {
               
                    dir('tests/bobcat') {
                       def mvnHome = tool 'M3'
	                sh "${mvnHome}/bin/mvn clean test -Dmaven.test.failure.ignore=true"
                    }
                
            } finally {
                junit testResults: 'tests/bobcat/target/*.xml', allowEmptyResults: true
                archiveArtifacts 'tests/bobcat/target/**'
            }
    
          
    }
    
  }
}

def dockerCmd(args) {
    sh "${DOCKER}/docker ${args}"
}

def getReleasedVersion() {
    return (readFile('pom.xml') =~ '<version>(.+)-SNAPSHOT</version>')[0][1]
}