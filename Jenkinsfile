pipeline {

    agent any

    // options {
    //     skipDefaultCheckout()
    // }

    tools {
        maven "maven"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.109.133.197:8081"
        NEXUS_REPOSITORY = "demo-release"
        NEXUS_REPO_ID = "demo-release"
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
        ARTVERSION = "${env.BUILD_ID}"
        TOMCAT_URL = "http://43.204.147.153:8080"
        TOMCAT_CREDENTIAL_ID = "tomcat_credentials"
    }

    stages {

        stage('BUILD') {
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

      // stage('CODE ANALYSIS with SONARQUBE') {
      //       environment {
      //           scannerHome = tool 'sonarscanner'
      //       }
      //       steps {
      //           withSonarQubeEnv('sonarserver') {
      //               sh '''${scannerHome}/bin/sonar-scanner \
      //                   -Dsonar.projectKey=vprofile \
      //                   -Dsonar.projectName=vprofile-repo \
      //                   -Dsonar.projectVersion=1.0 \
      //                   -Dsonar.sources=src/ \
      //                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
      //                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
      //                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
      //                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
      //           }
      //       }
      //   }

        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_URL}",
                        groupId: "com.visualpathit",
                        version: "v2",
                        repository: "${NEXUS_REPOSITORY}",
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: "vprofile",
                             classifier: '',
                             file: "target/vprofile-v2.war",
                             type: "war"],
                            [artifactId: "vprofile",
                             classifier: '',
                             file: "pom.xml",
                             type: "pom"]
                        ]
                    )
                }
            }
        }

        // stage('DEPLOY TO TOMCAT') {
        //     steps {
        //         script {
        //             withCredentials([usernamePassword(credentialsId: "${TOMCAT_CREDENTIAL_ID}", passwordVariable: 'TOMCAT_PASS', usernameVariable: 'TOMCAT_USER')]) {
        //                 sh '''
        //                 curl -u ${TOMCAT_USER}:${TOMCAT_PASS} -T target/vprofile-v2.war ${TOMCAT_URL}/manager/text/deploy?path=/vprofile&update=true
        //                 '''
        //             }
        //         }
        //     }
        // }
stage('DEPLOY TO TOMCAT') {
    steps {
        script {
            // Use the 'withCredentials' block to bind the credentials
            withCredentials([usernamePassword(credentialsId: "${TOMCAT_CREDENTIAL_ID}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                
                // Stop the app if it is running
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} ${TOMCAT_URL}/manager/text/undeploy?path=/vprofile
                """
                
                // Deploy the new WAR file to Tomcat
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} -T target/vprofile-v2.war ${TOMCAT_URL}/manager/text/deploy?path=/vprofile&update=true
                """
            }
        }
    }
}


    }
}
