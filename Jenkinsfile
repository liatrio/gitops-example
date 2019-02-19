def manifest
def environment
pipeline {
    agent any
    stages {
        stage('Deploy to Dev') {
            when {
                branch 'dev'
            }
            steps {

                 script {
                    manifest = readJSON file: 'manifest.json'
                    environment = readJSON file: 'dev-env.json'
                    echo "Deploying the manifest ${manifest.version} to DEV" //Get the artifact/image from an artifact repository
                    echo "Deploying WEB artifact ${manifest.artifacts.web} to DEV host ${environment.web.host}"
                    echo "Deploying API artifact ${manifest.artifacts.api} to DEV host ${environment.api.host}"
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                branch 'staging' 
            }
            steps {
                script {
                    manifest = readJSON file: 'manifest.json'
                    environment = readJSON file: 'staging-env.json'
                    echo "Deploying the manifest ${manifest.version} to STAGING" //Get the artifact/image from an artifact repository
                    echo "Deploying WEB artifact ${manifest.artifacts.web} to STAGING host ${environment.web.host}"
                    echo "Deploying API artifact ${manifest.artifacts.api} to STAGING host ${environment.api.host}"
                }
            }
        }
        stage('Deploy to Production') {
            when {
                branch 'production' 
            }
            steps {
                script {
                    manifest = readJSON file: 'manifest.json'
                    environment = readJSON file: 'prod-env.json'
                    echo "Deploying the manifest ${manifest.version} to PRODUCTION" //Get the artifact/image from an artifact repository
                    echo "Deploying WEB artifact ${manifest.artifacts.web} to PRODUCTION host ${environment.web.host}"
                    echo "Deploying API artifact ${manifest.artifacts.api} to PRODUCTION host ${environment.api.host}"
                }
            }
        }
        stage('Run Automated Tests') {
            steps {
                echo 'Running automated tests'
            }
        }
    }
}