COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline{
    agent any
    tools{
        maven "maven"
    }

    stages{
        stage ("stage:1 git clone the repo"){
            steps {
                sh "echo cloning the  repository..."
                //change to your git syntax you generated
                git branch: 'main', url: 'https://github.com/Iddrisu08/web-app.git' 
            }
        }

        stage("stage :2 build the application"){
            steps{
                sh "echo building the application"
                sh "mvn clean package"
            }
        }

        stage("stage: 3 test the application"){
            steps{
                sh "echo testing the application"
                sh "mvn test"
            }
        }

        stage("stage: 4 sonarqube scan"){
            environment{ 
                ScannerHome = tool "sonarqube" 
            }
            steps{
                script{
                    withSonarQubeEnv("sonarqube") //must match the soanrqube name configured in jenkins tools
                    {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=web-app" //projectkey is the name you'll to the project when deployed to sonarqube
                    }
                }
            }
        }

         stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
   
        stage("stage: 5 Nexus upload"){
            steps{
                // change to your nexus generated syntax
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/second-pipeline-job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '3.129.149.111:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'release', version: 'RELEASE'

            }
        }

        stage("stage: 6 deploy to tomcat"){
            steps{
                deploy adapters: [tomcat9(credentialsId: 'tomcat-id', path: '', url: 'http://18.116.24.58:8080/')], contextPath: null, war: 'target/*.war'
            }
        }

    }

    post {
        success {
            slackSend channel: 'team-usa', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: 'team-usa', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    }

} 