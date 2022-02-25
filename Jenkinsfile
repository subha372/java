pipeline {
    tools{
    git "git"
    maven "M3"
    }
    environment {
                // This can be nexus3 or nexus2
                NEXUS_VERSION = "nexus2"
                // This can be http or https
                NEXUS_PROTOCOL = "http"
                // Where your Nexus is running
                NEXUS_URL = "20.204.130.223:8081/repository/Maven-Nexus-repo/"
                // Repository where we will upload the artifact
                NEXUS_REPOSITORY = "Maven-Nexus-repo"
                // Jenkins credential id to authenticate to Nexus OSS
                NEXUS_CREDENTIAL_ID = "Nexus_credentials"
            }
        agent any
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/subha372/java.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('code-analysis') {
            environment {

                def scannerHome = tool 'sonar_scanner';
                PROJECT_NAME = "java_project_subha"
                }
            steps {
                
                withSonarQubeEnv('sonarqube') { // If you have configured more than one global server connection, you can specify its name
                sh '''$scannerHome/bin/sonar-scanner \
            -Dsonar.java.binaries=target/classes \
            -Dsonar.projectKey=$PROJECT_NAME \
            -Dsonar.sources=src \
            -Dsonar.projectVersion=1.0 \
            -Dsonar.projectName=$PROJECT_NAME \
            -Dsonar.sourceEncoding=UTF-8 \
            -Dsonar.language=java'''
                }
            }
        }


        stage('package') {
            steps {
                sh "mvn package"
            }
        }
        stage('Artifactory_upload') {
            
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],

                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }        
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploy'
            }
        }
    }
}
