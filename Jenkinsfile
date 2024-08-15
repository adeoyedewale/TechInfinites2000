pipeline {
    agent any
    tools {
        maven "Maven3"
        jdk "jdk8"
    }

    environment {
        
        NEXUS_VERSION = "nexus3"
        
        NEXUS_PROTOCOL = "http"
        
        NEXUS_URL = "10.204.15.170:8081" 
        
        // NEXUS_REPOSITORY = "thirdparty"
        
        NEXUS_CREDENTIAL_ID = "nexus-credential-id"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage("Check out") {
            steps {
                script {
                    git branch: 'main', url:'https://github.ec.va.gov/EPMO/tims-app.git', credentialsId: 'githubtoken'
                }
            }
        }

        stage("mvn build") {
            steps {
                script {
                    sh "mvn clean install -Dmaven.test.skip=true"
                }
            }
        }
        
        stage("Determine Nexus Repository") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    if (pom.version.endsWith("SNAPSHOT")) {
                        env.NEXUS_REPOSITORY = "thirdparty-snapshots"
                    } else if (pom.version.endwith ("RELEASE"))
                    {
                        env.NEXUS_REPOSITORY = "thirdparty-releases"
                    }else {
                        currentBuild.result = "FAILURE"
                        throw new Exception ("version must end with SNAPSHOT|RELEASE")
                    }
                    
                    
                    echo "Deploying to Nexus repository: ${env.NEXUS_REPOSITORY}"
                }
            }
        }

        stage("publish to nexus") {
            steps {
                script {
                    
                    pom = readMavenPom file: "pom.xml";
                    
                    filesByGlob = findFiles(glob: "**/target/*.jar");
                    
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    
                    artifactPath = filesByGlob[0].path;
                    
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: 'jar']
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
       
    }
}
