pipeline {
    agent any
    tools {
        maven "M3"
    }

    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "13.201.67.170:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "mymaven_repo"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "jenkins"
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage("Check out") {
            steps {
                script {
                    git 'https://github.com/suram6534/sparkjava-war-example.git'
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

        stage("publish to nexus") {
            steps {
                script {
                    // Read POM XML file using 'readMavenPom' step
                    pom = readMavenPom file: "pom.xml"
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path
                    // Assign to a boolean response verifying if the artifact name exists
                    artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear, and .war files.
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    } // ✅ Correctly closed 'stages' block
} // ✅ Correctly closed 'pipeline' block
