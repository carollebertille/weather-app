pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '7'))
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout (time: 1, unit: 'MINUTES')
        timestamps()
    }
    environment {
        registry = '801455127377.dkr.ecr.us-east-1.amazonaws.com/images'
        dockerimage = '' 
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string (name: 'APP_NAME', defaultValue: 'weather', description: '')
    }
    stages {
        stage ('Checkout') {
            steps {
                dir("${WORKSPACE}/weather-code") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${env.BRANCH_NAME}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch']],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                            url: 'git@github.com:carollebertille/weather-app.git',
                            credentialsId: 'github-auth'
                        ]]
                    ])
                }
            }
        }
        /*stage('SonarQube analysis') {
            agent {
                docker {
                 image 'sonarsource/sonar-scanner-cli:4.8'
                }
            }
               environment {
                 CI = 'true'
                  scannerHome='/opt/sonar-scanner'
            }
             steps{
               withSonarQubeEnv('sonar-scanner') {
                 sh "${scannerHome}/bin/sonar-scanner"
                }
              }
            }*/
        stage('SonarQube analysis') {
            steps {
                script {
                    def sourceCodeDir = "${WORKSPACE}/weather-code/application/${params.APP_NAME}"
                    dir(sourceCodeDir) {
                        docker.image('sonarsource/sonar-scanner-cli:5').inside {
                            withSonarQubeEnv('sonar-scanner') {
                                sh "ls"
                                sh "/opt/sonar-scanner/bin/sonar-scanner"
                            }
                        }
                    }
                }
            }
        }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        stage('Pushing to ECR') {
            steps {
                script {
                    // Authenticate with AWS ECR
                    docker.withRegistry("https://"+registry,"ecr:us-east-1:"+registryCredential) { 
                      dockerImage.push()
                  }
            }
          }
        }
    }
}
