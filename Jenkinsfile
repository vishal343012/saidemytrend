def registry = 'https://trialkub2kx.jfrog.io'

pipeline {                                    // 1  // Defines the start of the Jenkins pipeline block

    agent any                                 // Specifies the pipeline can run on any available agent

    environment {                             // 2  // Defines environment variables for the pipeline
        PATH = "/opt/maven/bin:$PATH"         // Adds Maven's path to the system's PATH variable
    }                                         // 2  // Ends the environment block

    stages {                                  // 3  // Defines the stages block where multiple stages are declared

        stage("build") {                      // 4  // Creates a stage named 'build'
            steps {                           // 5  // Defines the steps that will be executed in this stage
                echo "----------- build started ----------"  
                                              // Logs a message indicating the start of the build
                sh 'mvn clean deploy'
                                              // Runs Maven clean and deploy commands, skipping tests
                echo "----------- build completed ----------"  
                                              // Logs a message indicating the build completion
            }                                 // 5  // Ends the steps block for 'build' stage
        }                                     // 4  // Ends the 'build' stage

        stage("test") {                       // 6  // Creates a stage named 'test'
            steps {                           // 7  // Defines the steps that will be executed in this stage
                echo "----------- unit test started ----------"  
                                              // Logs a message indicating the start of unit tests
                sh 'mvn surefire-report:report'  
                                              // Runs the Maven Surefire report to execute unit tests
                echo "----------- unit test completed ----------"  
                                              // Logs a message indicating unit test completion
            }                                 // 7  // Ends the steps block for 'test' stage
        }                                     // 6  // Ends the 'test' stage

        stage('SonarQube analysis') {         // 8  // Creates a stage named 'SonarQube analysis'
            environment {                     // 9  // Defines environment variables specific to this stage
                scannerHome = tool 'saidemy-sonar-scanner'  
                                              // Sets the SonarQube scanner tool
            }                                 // 9  // Ends the environment block for this stage

            steps {                           // 10  // Defines the steps that will be executed in this stage
                withSonarQubeEnv('saidemy-sonarqube-server') {  
                                              // Executes the SonarQube analysis within the SonarQube environment
                    sh "${scannerHome}/bin/sonar-scanner"  
                                              // Runs the SonarQube scanner tool
                }                             // Ends the withSonarQubeEnv block
            }                                 // 10  // Ends the steps block for 'SonarQube analysis' stage
        }                                     // 8  // Ends the 'SonarQube analysis' stage

        stage("Jar Publish") {                // 14  // Creates a stage named 'Jar Publish'
            steps {                           // 15  // Defines the steps that will be executed in this stage
                script {                      // 16  // Allows running custom Groovy script inside the pipeline
                    echo '<--------------- Jar Publish Started --------------->'  
                                              // Logs a message indicating the start of JAR publishing
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifact-cred"  
                                              // Defines the Artifactory server with the specified URL and credentials
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"  
                                              // Sets properties like build ID and Git commit ID for the build
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "sai-libs-release-local/{1}",
                              "flat": "false",
                              "props": "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""  
                                              // Defines the upload specification for uploading JAR files to Artifactory
                    def buildInfo = server.upload(uploadSpec)  
                                              // Uploads the files to Artifactory and collects build info
                    buildInfo.env.collect()  
                                              // Collects environment variables as part of the build info
                    server.publishBuildInfo(buildInfo)  
                                              // Publishes the build information to Artifactory
                    echo '<--------------- Jar Publish Ended --------------->'  
                                              // Logs a message indicating the end of JAR publishing
                }                             // 16  // Ends the script block for 'Jar Publish' stage
            }                                 // 15  // Ends the steps block for 'Jar Publish' stage
        }                                     // 14  // Ends the 'Jar Publish' stage

    }                                         // 3  // Ends the stages block
}                                             // 1  // Ends the pipeline block

