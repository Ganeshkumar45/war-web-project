pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "3.109.133.197:8081"
        NEXUS_REPOSITORY = "demo-release"
        NEXUS_CREDENTIAL_ID = "nexus_credentials"
        ARTVERSION = "1.0.0"
        TOMCAT_URL = "http://43.204.147.153:8080"
        TOMCAT_CREDENTIAL_ID = "tomcat_credentials"
        TOMCAT_USERNAME = "tomcat-user"
        TOMCAT_PASSWORD = "secure-password" // Update this to the correct password
    }

    stages {
        stage('Build WAR') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.war'
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def warFile = sh(script: 'find target -name "*.war" -print -quit', returnStdout: true).trim()
                    if (!fileExists(warFile)) {
                        error("WAR file not found at ${warFile}")
                    }
                    nexusArtifactUploader(
                        nexusVersion: "${NEXUS_VERSION}",
                        protocol: "${NEXUS_PROTOCOL}",
                        nexusUrl: "${NEXUS_URL}",
                        groupId: "com.example.warwebproject",
                        version: "${ARTVERSION}",
                        repository: "${NEXUS_REPOSITORY}",
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: "wwp", classifier: '', file: warFile, type: "war"],
                            [artifactId: "wwp", classifier: '', file: "pom.xml", type: "pom"]
                        ]
                    )
                }
            }
        }

stage('Deploy WAR') {
    steps {
        script {
            // Dynamically find the latest WAR file in the target directory
            def warFilePath = sh(script: "find ${workspace}/target -name '*.war' -type f -print0 | xargs -0 ls -t | head -n 1", returnStdout: true).trim()

            // Check if the WAR file exists
            if (warFilePath) {
                // Debug: Output the found WAR file path
                echo "WAR file located at: ${warFilePath}"
                sh "ls -l ${warFilePath}"  // Verify WAR file permissions

                // SSH into the remote server and deploy WAR file
                sh """
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/jenkins_key ubuntu@43.204.147.153 << EOF
                        # Debug: Print system info
                        echo "Deploying new WAR file..."
                        
                        # Ensure webapps directory has the right permissions
                        sudo chmod -R 777 /opt/tomcat/webapps/
                        
                        # Copy WAR file using scp (using absolute path for the WAR file)
                        scp -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/jenkins_key ${warFilePath} ubuntu@43.204.147.153:/opt/tomcat/webapps/wwp.war
                        
                        # Restart Tomcat
                        sudo systemctl restart tomcat
                        
                        # Set proper permissions after restart
                        sudo chmod -R 755 /opt/tomcat/webapps/
                        
                        echo "Deployment complete."
                    EOF
                """
            } else {
                error "No WAR file found to deploy."
            }
        }
    }
}

    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed. Check logs for details.'
        }
    }
}
