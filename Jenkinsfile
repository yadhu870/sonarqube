pipeline {
    agent any
    tools {
        maven "maven"
        jdk "java"
    }

    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "172.31.7.76:8080"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "yadhu-auto"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage("Check out") {
            steps {
                script {
                    git 'https://github.com/yadhu870/jenkins-test.git'
                }
            }
        }

        stage("mvn build") {
            steps {
                script {
                    sh "mvn clean package"
                }
            }
        }
stage('SonarQube analysis') {
//    def scannerHome = tool 'SonarScanner 4.0';
        steps{
        withSonarQubeEnv('sonarqube') {
        // If you have configured more than one global server connection, you can specify its name
//      sh "${scannerHome}/bin/sonar-scanner"
        sh "mvn sonar:sonar"
    }
        }
        }
        stage("publish to nexus") {
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

                         nexusArtifactUploader artifacts: [
                        [
                            artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging
                            ]
                            ],
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            groupId: pom.groupId,
                            nexusUrl: NEXUS_URL,
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            repository: NEXUS_REPOSITORY,
                            version: '${BUILD_NUMBER}'


                    } else
                    {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage("deploy to tomcat"){
                     steps{
                          sshagent(['a4a074d6-0481-4476-a01a-1b36f1829f1e'])
                          {
               sh 'scp -o StrictHostKeyChecking=no target/webapp.war root@172.31.2.193:/opt/tomcat/webapps/'
                           }
                          }
                     }

               }
         }

       

