pipeline {
    agent any
    tools{
        jdk 'javaJDK17'
        maven 'maven3'
    }
    
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Branch to Build')
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Where to Deploy?')
    }
    
    environment {
        AWS_REGION = "us-east-1"
        S3_BUCKET = "apps.spontansolutions"
    }

    stages{
        stage('GIT Checkout'){
            steps{
                git branch: '$BRANCH_NAME', url: 'https://github.com/kosaraju3333/bank-app.git'
            }
        }
        stage('Build'){
            steps{
                sh 'cd app && mvn clean package -DskipTests'
            }
        }
        stage('Push artifact to AWS S3'){
            steps {
                script {
                    // Generate REAL TIME at the moment of build
                    def build_time = sh(script: "TZ='Asia/Kolkata' date +%d-%m-%Y-%H:%M:%S", returnStdout: true).trim()
                    def version = "V-${build_time}"
                    
                    // Save it globally so other stages can use it
                    env.APP_VERSION = version
                    echo "Building VERSION = ${env.APP_VERSION}"
                }
                sh '''
                    echo "Uploading artifact to S3..."
                    aws s3 cp app/target/bankapp-*.jar s3://$S3_BUCKET/bank-app-artifacts/$BRANCH_NAME/
                    aws s3 cp app/target/bankapp-*.jar s3://$S3_BUCKET/bank-app-artifacts/$BRANCH_NAME/bank-app-$BRANCH_NAME-${APP_VERSION}.jar
                '''
            }
        }
        stage('Deploy') {
            when { expression { return params.DEPLOY_ENV != '' } }
            steps {
                script {
                    if (params.DEPLOY_ENV == "dev") {
                        sh "ssh ubuntu@dev-server 'sudo systemctl restart app'"
                    }
                    if (params.DEPLOY_ENV == "staging") {
                        input message: "Deploy to staging?"
                        sh "ssh ubuntu@staging-server 'sudo systemctl restart app'"
                    }
                    if (params.DEPLOY_ENV == "prod") {
                        input message: "Deploy to PRODUCTION?"
                        sh "ssh ubuntu@prod-server 'sudo systemctl restart app'"
                    }
                }
            }
        }
    }
}
