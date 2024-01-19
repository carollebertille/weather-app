pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '2'))
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout (time: 1, unit: 'MINUTES')
        timestamps()
    }
    environment {
        registry = '801455127377.dkr.ecr.us-east-1.amazonaws.com/images'
        registryCredential = 'jenkins-ecr'
        
    }
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string (name: 'APP_NAME', defaultValue: 'weather', description: '')
        string(name: 'auth-tag',  defaultValue: '0.0.0',   description: '')
        string(name: 'ui-tag',  defaultValue: '0.0.0',   description: '')
        string(name: 'redis-tag',  defaultValue: '0.0.0',   description: '')
        string(name: 'weather-tag',  defaultValue: '0.0.0',   description: '')
        string(name: 'db-tag',  defaultValue: '0.0.0',   description: '')
    }
    stages {
        stage ('Checkout') {
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
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
        /*stage('SonarQube Analysis') {
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                        withSonarQubeEnv('sonar-scanner') {
                            sh "/var/opt/sonar-scanner/bin/sonar-scanner"
                        }
                    }
                }
            }
        }*/
        stage('build images auth') {
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/auth
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/images:${auth-tag} . 
                            cd ../code-dockerfile/UI
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/images:${ui-tag} . 
                            cd ../code-dockerfile/Redis
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/images:${redis-tag} . 
                            cd ../code-dockerfile/DB
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/images:${db-tag} . 
                            cd ../code-dockerfile/weather
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/images:${weather-tag} . 
                            '''
                    }
                }
        }
    }
 }
}        
