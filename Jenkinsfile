pipeline {
    agent any

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "65.2.143.128:8081" // Updated Nexus URL
        NEXUS_REPOSITORY = "maven-snapshots" // Updated repository name
        NEXUS_REPO_ID = "maven-snapshots"
        NEXUS_CREDENTIAL_ID = "nexus_credentials" // Updated credential ID
        ARTVERSION = "${env.BUILD_ID}" // Artifact version as Build ID
        SONAR_HOST_URL = "http://3.109.186.241:9000" // SonarQube server URL
        SONAR_PROJECT_KEY = "org.springframework:gs-maven"
        SONAR_PROJECT_NAME = "gs-maven"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Build completed successfully. Archiving artifacts...'
                    archiveArtifacts artifacts: '**/target/*.war, **/target/*.jar'
                }
            }
        }


        stage('Code Analysis with Checkstyle') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Checkstyle analysis completed successfully.'
                }
            }
        }

        stage('Code Analysis with SonarQube') {
            environment {
                scannerHome = tool 'sonarscanner4'
            }
            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                            -Dsonar.projectVersion=1.0 \
                            -Dsonar.sources=src/ \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.junit.reportsPath=target/surefire-reports/ \
                            -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    """
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.war") + findFiles(glob: "target/*.jar")

                    if (filesByGlob.isEmpty()) {
                        error "No WAR or JAR artifact found in the target directory!"
                    }

                    filesByGlob.each { file ->
                        def artifactPath = file.path
                        echo "Found artifact: ${artifactPath}, Group ID: ${pom.groupId}, Packaging: ${pom.packaging}, Version: ${pom.version}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                 classifier: '',
                                 file: artifactPath,
                                 type: pom.packaging],
                                [artifactId: pom.artifactId,
                                 classifier: '',
                                 file: "pom.xml",
                                 type: "pom"]
                            ]
                        )
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
