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
        CREDENTIALSAWS = credentials('aws-credentials')
        DOCKERHUB_REGISTRY = "edennolsn2021"
        ECR_REGISTRY_URI = "801455127377.dkr.ecr.us-east-1.amazonaws.com"
        UI_ECR_REPOSITORY_NAME = "weather-ui"
        AUTH_ECR_REPOSITORY_NAME = "weather-auth"
        WEATHER_ECR_REPOSITORY_NAME = "weather-weather"
        REDIS_ECR_REPOSITORY_NAME = "weather-redis"
        DB_ECR_REPOSITORY_NAME = "weather-db" 
    }
    parameters {
        choice(
            choices: ['dockerhub', 'ecr'], 
            name: 'Registry'
          )
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: '')
        string(name: 'DB_IMAGE_TAG', defaultValue: '0.0.0', description: '')
        string(name: 'REDIS_IMAGE_TAG', defaultValue: '0.0.0', description: '')
        string(name: 'UI_IMAGE_TAG', defaultValue: '0.0.0', description: '')
        string(name: 'WEATHER_IMAGE_TAG', defaultValue: '0.0.0', description: '')
        string(name: 'AUTH_IMAGE_TAG', defaultValue: '0.0.0', description: '')
        string(name: 'APP_NAME', defaultValue: 'weather', description: '')
        string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: '')
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
        stage('build all images (auth, ui, redis, weather, db)') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                dir("${WORKSPACE}/app-code/application/${params.APP_NAME}") {
                    script {
                         sh """
                            cd code-dockerfile/auth
                            docker build -t ${ECR_REGISTRY_URI}/${AUTH_ECR_REPOSITORY_NAME}:${params.AUTH_IMAGE_TAG} .
                            cd ../../code-dockerfile/UI
                            docker build -t ${ECR_REGISTRY_URI}/${UI_ECR_REPOSITORY_NAME}:${params.UI_IMAGE_TAG} .
                            cd ../../code-dockerfile/DB
                            docker build -t ${ECR_REGISTRY_URI}/${DB_ECR_REPOSITORY_NAME}:${params.DB_IMAGE_TAG} .
                            cd ../../code-dockerfile/Redis
                            docker build -t ${ECR_REGISTRY_URI}/${REDIS_ECR_REPOSITORY_NAME}:${params.REDIS_IMAGE_TAG} .
                            cd ../../code-dockerfile/weather
                            docker build -t ${ECR_REGISTRY_URI}/${WEATHER_ECR_REPOSITORY_NAME}:${params.WEATHER_IMAGE_TAG} .
                            """
                    }
                }
        }
    }
    stage('Getting AWS Credentials') {
            steps {
                script {
                   def awsCredentialsId = 'aws-credentials'
                   withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        credentialsId: awsCredentialsId,
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        aws_credentials()

                        sh """
                            aws s3 ls
                        """
                   } 
                }
            }
        }
     stage('Login and Push into ecr') {
            when{  
            expression {
              params.Registry == 'ecr' }
              }
            steps {
                    script {
                         sh """
                            aws ecr get-login-password --region ${params.AWS_REGION} | sudo docker login --username AWS --password-stdin ${ECR_REGISTRY_URI}
                            docker push ${ECR_REGISTRY_URI}/${UI_ECR_REPOSITORY_NAME}:${params.UI_IMAGE_TAG}
                            docker push ${ECR_REGISTRY_URI}/${AUTH_ECR_REPOSITORY_NAME}:${params.AUTH_IMAGE_TAG}
                            docker push ${ECR_REGISTRY_URI}/${WEATHER_ECR_REPOSITORY_NAME}:${params.WEATHER_IMAGE_TAG}
                            docker push ${ECR_REGISTRY_URI}/${REDIS_ECR_REPOSITORY_NAME}:${params.DB_IMAGE_TAG}
                            docker push ${ECR_REGISTRY_URI}/${DB_ECR_REPOSITORY_NAME}:${params.DB_IMAGE_TAG}
                            """
                        
                    }
          }
      } 
      /*stage('Push all images  to ecr') {
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
      }*/ 
   
            
 }
/* post {
         success {
             slackSend color: '#2EB67D',
             channel: '#develop', 
             message: "*Alpha Project Build Status*" +
             "\n Project Name: Weather" +
             "\n Job Name: ${env.JOB_NAME}" +
             "\n Build number: ${currentBuild.displayName}" +
             "\n Build Status : *SUCCESS*" +
             "\n Build url : ${env.BUILD_URL}"
         }
         failure {
             slackSend color: '#E01E5A',
             channel: '#develop',  
             message: "*Weather Project Build Status*" +
             "\n Project Name: Weather" +
             "\n Job Name: ${env.JOB_NAME}" +
             "\n Build number: ${currentBuild.displayName}" +
             "\n Build Status : *FAILED*" +
             "\n Action : Please check the console output to fix this job IMMEDIATELY" +
             "\n Build url : ${env.BUILD_URL}"
         }
         unstable {
             slackSend color: '#ECB22E',
             channel: '#develop', 
             message: "*Weather Project Build Status*" +
             "\n Project Name: Weather" +
             "\n Job Name: ${env.JOB_NAME}" +
             "\n Build number: ${currentBuild.displayName}" +
             "\n Build Status : *UNSTABLE*" +
             "\n Action : Please check the console output to fix this job IMMEDIATELY" +
             "\n Build url : ${env.BUILD_URL}"
         }   
     }*/
} 
def aws_credentials() {
sh """    
mkdir -p $HOME/.aws || true

cat <<EOF >  $HOME/.aws/credentials
[default]
aws_access_key_id = ${AWS_ACCESS_KEY_ID}
aws_secret_access_key = ${AWS_SECRET_ACCESS_KEY}
EOF

cat <<EOF >  $HOME/.aws/config
[default]
region = ${params.AWS_REGION}
output = json
EOF
"""
}
