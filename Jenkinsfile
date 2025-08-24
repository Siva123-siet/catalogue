pipeline {
    // Pre-build
    agent {
        label 'AGENT-1'
    }
    environment { 
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "097003440739"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds() 
    }
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    // Build 
    stages {
        stage('Read package.json') {
            steps {
                script {
                    // Read the package.json file
                    def packageJson = readJSON file: 'package.json'
                    // Access the version field
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }
        stage('Install dependencies') {
            steps {
                script {
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Unit Testing') {
            steps {
                script {
                    sh """
                        echo "unit tests"
                    """
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                  withAWS(credentials: 'aws-auth', region: 'us-east-1') {
                // Your AWS CLI commands or other AWS interactions go here
                sh """
                  aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                  docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                  docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                """
                }
                }
            }
        }
        stage('Trigger deploy') {
            when {
                    expression { params.deploy }
                }
            steps {
                script {
                    build job: 'catalogue-cd',
                    parameters [
                     string(name: 'appVersion', value: "${appVersion}"),
                     string(name: 'deploy_to', value: 'dev')
                    ],
                    propogate: false, //even sg fails vpc will not be effected
                    wait: false // vpc will not wait for sg pipeline completion
                }
            }
        }
    }
    //post build
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello failure'
        }
    }
}