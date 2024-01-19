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
        stage('Remove Existing sonar-project.properties') {
            steps {
                dir("${WORKSPACE}/code/application/${params.APP_NAME}") {
                    script {
                        // Check if sonar-project.properties exists and remove it if found
                        if (fileExists('sonar-project.properties')) {
                            sh 'rm sonar-project.properties'
                        }
                    }
                }
            }
        }
        stage('Create sonar-project.properties') {
            steps {
                dir("${WORKSPACE}/weather-code/application/${params.APP_NAME}") {
                    script {
                        // Define the content of sonar-project.properties
                        def sonarProjectPropertiesContent = """
                            sonar.host.url=https://sonarqube.ektechsoftwaresolution.com/
                            sonar.projectKey=weather-code
                            sonar.projectName=weather-code
                            sonar.projectVersion=1.0
                            sonar.sources=code-dockerfile
                            qualitygate.wait=true
                        """

                        // Create the sonar-project.properties file
                        writeFile file: 'sonar-project.properties', text: sonarProjectPropertiesContent
                    }
                }
            }
        }
        stage('Open sonar-project.properties') {
            steps {
                dir("${WORKSPACE}/weather-code/application/${params.APP_NAME}") {
                    script {
                        // Use 'cat' command to display the content of sonar-project.properties
                        sh 'cat sonar-project.properties'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir("${WORKSPACE}/weather-code/application/${params.APP_NAME}") {
                    script {
                        withSonarQubeEnv('Sonar-scanner') {
                            sh "/var/opt/sonar-scanner/bin/sonar-scanner"
                        }
                    }
                }
            }
        }
    }
}
