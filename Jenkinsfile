def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    environment{
        //MESSAGE='Hello Folks'
        WORKSPACE = "${env.WORKSPACE}"
        //BUILD_NUMBER = '${env.BUILD_NUMBER}'   
    }
    
    tools{
        maven 'localMaven'
        jdk 'localJDK'
    }
    
    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/damioneh/Jenkins-CI-CD-Pipeline-Project.git'
                sh 'mvn --version'
                echo '${env.BUILD_NUMBER}, ${WORKSPACE}'
            }
        }
        
        stage('Build') {
            steps {
                sh 'java -version'
                //sh 'mvn clean package'
                sh 'mvn package'
                echo '${MESSAGE}'
            }
            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        
        stage('Unit Test'){
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        
        stage ('SonarQube scanning'){
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=JavaWebApp \
                      -Dsonar.host.url=http://172.31.17.55:9000 \
                      -Dsonar.login=5cb91ba8944fab5fa0fbf46402a00cbd5909f107
                    """
                }
            }
        }
    
        stage("Quality Gate"){
            steps{
                waitForQualityGate abortPipeline: true
            }
        }
    
        stage("Upload artifact to Nexus"){
            steps{
                sh 'mvn clean deploy -DskipTests'
            }
        }
        
        stage('Deploy to DEV') {
            environment {
                HOSTS = "dev"
            }
            steps {
                sh 'ls'
                sh "ansible-playbook $WORKSPACE/deploy.yaml --extra-vars \'hosts=$HOSTS workspace_path=$WORKSPACE\'"
            }
        }
        
        stage('Approval') {
            steps {
                input('Do you want to proceed?')
            }
        }
        
        stage('Deploy to STAGE') {
            environment {
                HOSTS = "stage"
            }
            steps {
                sh 'ls'
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \'hosts=$HOSTS workspace_path=${WORKSPACE}\'"
            }
        }
        
        stage('Deploy to PROD') {
            environment {
                HOSTS = "prod"
            }
            steps {
                sh 'ls'
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \'hosts=$HOSTS workspace_path=${WORKSPACE}\'"
            }
        
            post {
                always {
                    echo 'I will always say Hello again!'
                    slackSend channel: '#glorious-w-f-devops-alerts', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
                }
            }
        }
    }
}