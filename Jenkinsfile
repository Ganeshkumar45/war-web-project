pipeline {

    agent any

    tools {
        maven "maven"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.109.133.197:8081"
        NEXUS_REPOSITORY = "demo-release"  // Using demo-release
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
        ARTVERSION = "1.0.0"  // Static version for release
        TOMCAT_URL = "http://43.204.147.153:8080"
        TOMCAT_CREDENTIAL_ID = "tomcat_credentials"
    }

    stages {
        stage('Build WAR') {
            steps {
                // Clean and package the WAR file
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    echo 'Archiving artifacts...'
                    // Archive WAR file generated in the target directory
                    archiveArtifacts artifacts: '**/target/*.war'  // Will archive any WAR file in the target folder
                }
            }
        }

        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    // Dynamically find the WAR file in the target directory
                    def warFile = sh(script: 'find target -name "*.war" -print -quit', returnStdout: true).trim()
                    echo "WAR file to be uploaded: ${warFile}"

                    // Upload WAR file and POM file to Nexus repository
                    nexusArtifactUploader(
                        nexusVersion: "${NEXUS_VERSION}",
                        protocol: "${NEXUS_PROTOCOL}",
                        nexusUrl: "${NEXUS_URL}",
                        groupId: "com.example.warwebproject",
                        version: "${ARTVERSION}",  // Static version for release
                        repository: "demo-release",  // Changed to demo-release
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: "wwp",  // Updated artifactId
                             classifier: '',
                             file: warFile,  // Dynamically picked WAR file
                             type: "war"],
                            [artifactId: "wwp",  // Updated artifactId
                             classifier: '',
                             file: "pom.xml",
                             type: "pom"]
                        ]
                    )
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    // Dynamically find the WAR file in the target directory
                    def warFile = sh(script: 'find target -name "*.war" -print -quit', returnStdout: true).trim()
                    echo "Deploying WAR file: ${warFile}"

                    withCredentials([usernamePassword(credentialsId: "${TOMCAT_CREDENTIAL_ID}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        // Undeploy existing app from Tomcat
                        sh """
                            curl -u ${TOMCAT_USER}:${TOMCAT_PASS} ${TOMCAT_URL}/manager/text/undeploy?path=/wwp
                        """
                        
                        // Deploy new WAR file to Tomcat
                        sh """
                            curl -u ${TOMCAT_USER}:${TOMCAT_PASS} -T ${warFile} ${TOMCAT_URL}/manager/text/deploy?path=/wwp&update=true
                        """
                    }
                }
            }
        }

    }
}
