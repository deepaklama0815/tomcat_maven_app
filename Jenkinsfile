pipeline{
    agent none

    environment{
        VM_USER = "ec2-user"
        VM_SERVER = "172.31.81.98" //target public ip
        TAG = "joedeep09/tomcat"  //docker hub repo
        PORT = 8080

    }
    parameters{
        choice(choices: 'Yes\nNo', description: 'Please choose the environment to deploy', name: 'Deploy')
        choice(choices: 'Yes\nNo', description: 'Deploy to Docker', name: 'DockerDeploy')
    }
    stages{
        stage('Junit'){
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8'
                    label 'maven'
                }
            }
            steps{
                sh 'mvn test verify' // --settings settings.xml test verify surefire-report:report-only'
            }
            post{
                success{
                   // junit "target/surefire-reports/*.xml"
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site', reportFiles: 'surefire-report.html', reportName: 'SureFireReportHTML', reportTitles: ''])

                }
            }
        }
        stage('build'){
            agent{
                dockerfile{
                    filename 'Dockerfile-project'
                    label 'maven'
                }
            }
            stages{
                stage('read the POM'){
                    steps{
                        script{
                            pom = readMavenPom file: 'pom.xml'
                            ARTIFACT_NAME = pom.artifactId.toString()
                        }
                    }
                }
                stage('packaging'){
                    steps{
                        sh 'mvn deploy --settings settings.xml -DSkipTests=True'
                        // stash includes: "target/${ARTIFACT_NAME}.war", name: 'artifact'
                    }
                    post{
                        success{
                            echo "Package has been uploaded to artifactory successfully"
                        }
                    }
                }
            }
        }
    }
}