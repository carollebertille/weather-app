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
        registryCredential = 'jenkins-ecr'
        REGISTRY = "801455127377.dkr.ecr.us-east-1.amazonaws.com"
        REGION = "us-east-1"
    }
    parameters {
        choice(
            choices: ['dockerhub', 'ecr'], 
            name: 'Registry'
          )
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string(name: 'DB_IMAGE_VERSION', defaultValue: '0.0.0', description: '')
        string(name: 'REDIS_IMAGE_VERSION', defaultValue: '0.0.0', description: '')
        string(name: 'UI_IMAGE_VERSION', defaultValue: '0.0.0', description: '')
        string(name: 'WEATHER_IMAGE_VERSION', defaultValue: '0.0.0', description: '')
        string(name: 'AUTH_IMAGE_VERSION', defaultValue: '0.0.0', description: '')
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
        stage('build images') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/auth
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/auth:${AUTH_IMAGE_VERSION} . 
                            cd ../../code-dockerfile/UI
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/ui:${UI_IMAGE_VERSION} . 
                            cd ../../code-dockerfile/Redis
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/redis:${REDIS_IMAGE_VERSION} . 
                            cd ../../code-dockerfile/weather
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/weather:${WEATHER_IMAGE_VERSION} . 
                            cd ../../code-dockerfile/DB
                            docker build -t 801455127377.dkr.ecr.us-east-1.amazonaws.com/db:${DB_IMAGE_VERSION} . 
                            '''
                    }
                }
        }
    }
    stage('Login ecr') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                    script {
                         sh '''
                            aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REGISTRY/ui
                            aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REGISTRY/auth
                            aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REGISTRY/weather
                            aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REGISTRY/db
                            aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $REGISTRY/redis
                            '''
                    }
                
        }
    }
   
            
 }
}        
