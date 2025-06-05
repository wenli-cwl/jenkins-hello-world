pipeline {
    agent any
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'qa', 'uat', 'prod'], description: 'Deployment environment')
    }
    environment {
        BUILD_DIR = 'build'
        PACKAGE_NAME = 'hello-world'
        GITHUB_TOKEN = credentials('github-token')
    }
    stages {
        stage('Setup Environment') {
            steps {
                script {
                    switch (params.DEPLOY_ENV) {
                        case 'dev':
                            env.DEPLOY_PORT = env.DEV_DEPLOY_PORT
                            env.SITE_NAME = env.DEV_SITE_NAME
                            break
                        case 'qa':
                            env.DEPLOY_PORT = env.QA_DEPLOY_PORT
                            env.SITE_NAME = env.QA_SITE_NAME
                            break
                        case 'uat':
                            env.DEPLOY_PORT = env.UAT_DEPLOY_PORT
                            env.SITE_NAME = env.UAT_SITE_NAME
                            break
                        default:
                            env.DEPLOY_PORT = env.PROD_DEPLOY_PORT
                            env.SITE_NAME = env.PROD_SITE_NAME
                    }
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    bat 'mkdir -p $BUILD_DIR'
                    bat 'cp index.html $BUILD_DIR/'
                }
            }
        }
        stage('Test') {
            steps {
                bat 'echo "<testsuite><testcase classname=\"hello\" name=\"world\"/></testsuite>" > test-results.xml'
                junit 'test-results.xml'
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: "$BUILD_DIR/**/*", fingerprint: true
            }
        }
        stage('Approval') {
            steps {
                script {
                    def msg = "Approve deployment to ${params.DEPLOY_ENV} on port ${env.DEPLOY_PORT}?"
                    input message: msg
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'iis-creds', usernameVariable: 'IIS_USER', passwordVariable: 'IIS_PASS')]) {
                    bat '''
                    powershell -NoProfile -Command "
                        Import-Module WebAdministration;
                        $siteName = env:SITE_NAME;
                        $physicalPath = env:WORKSPACE + '\\' + env:BUILD_DIR;
                        if (-not (Get-Website -Name $siteName -ErrorAction SilentlyContinue)) {
                            New-Website -Name $siteName -Port $env:DEPLOY_PORT -PhysicalPath $physicalPath -Force;
                        } else {
                            Set-ItemProperty IIS:\\Sites\\$siteName -Name physicalPath -Value $physicalPath;
                        }
                    "
                    '''
                }
            }
        }
        stage('Publish') {
            steps {
                bat 'echo "Publishing to GitHub Packages using token $GITHUB_TOKEN"'
            }
        }
    }
    post {
        success {
            mail to: 'wenli.choong@isatec.com', subject: 'Build and Deploy Successful', body: 'The deployment to IIS has finished.'
        }
    }
}
