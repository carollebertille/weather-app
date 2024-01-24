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
        UI_ECR_IMAGE_REPOSITORY_NAME = "ui"
        AUTH_ECR_REPOSITORY_NAME = "auth"
        WEATHER_ECR_REPOSITORY_NAME = "weather"
        REDIS_ECR_REPOSITORY_NAME = "redis"
        DB_ECR_REPOSITORY_NAME = "db"
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
        stage('build auth image') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/auth
                            docker build -t $REGISTRY/auth:${AUTH_IMAGE_VERSION} . 
                            '''
                    }
                }
        }
    }
    stage('build ui image') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/UI
                            docker build -t $REGISTRY/auth:${UI_IMAGE_VERSION} . 
                            '''
                    }
                }
        }
    }   
    stage('build DB image') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/DB
                            docker build -t $REGISTRY/auth:${DB_IMAGE_VERSION} . 
                            '''
                    }
                }
        }
    }
     stage('build redis image') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/redis
                            docker build -t $REGISTRY/auth:${REDIS_IMAGE_VERSION} . 
                            '''
                    }
                }
        }
    }
     stage('build weather image') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh '''
                            cd code-dockerfile/weather
                            docker build -t $REGISTRY/auth:${WEATHER_IMAGE_VERSION} . 
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
