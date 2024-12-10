pipeline {
    
	agent any
	    options {
        skipDefaultCheckout()
    }
	
	tools {
        maven "maven"
	
    }
	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "65.2.143.128:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	NEXUS_REPO_ID    = "vprofile-release"
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
        ARTVERSION = "${env.BUILD_ID}"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	// stage('UNIT TEST'){
 //            steps {
 //                sh 'mvn test'
 //            }
 //        }

	// stage('INTEGRATION TEST'){
 //            steps {
 //                sh 'mvn verify -DskipUnitTests'
 //            }
 //        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonarscanner'
          }

          steps {
            withSonarQubeEnv('sonarserver') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            // timeout(time: 10, unit: 'MINUTES') {
            //    waitForQualityGate abortPipeline: true
            // }
          }
        }

stage("Publish to Nexus Repository Manager") {
    steps {
        script {
            try {
                pom = readMavenPom file: "pom.xml"
                echo "POM File Read: ${pom.groupId}:${pom.artifactId}:${pom.version}"
                filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                if (filesByGlob.length == 0) {
                    error "No artifacts found in the target directory!"
                }
                artifactPath = filesByGlob[0].path
                artifactExists = fileExists artifactPath
                if (artifactExists) {
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: ARTVERSION,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                            [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                        ]
                    )
                } else {
                    error "Artifact not found: ${artifactPath}"
                }
            } catch (Exception e) {
                echo "Error during pipeline execution: ${e.message}"
                throw e
            }
        }
    }
}



    }


}
