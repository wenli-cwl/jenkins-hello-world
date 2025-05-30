pipeline {
    agent any
    environment {
        BUILD_DIR = 'build'
        PACKAGE_NAME = 'hello-world'
        GITHUB_TOKEN = credentials('github-token')
    }
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'mkdir -p $BUILD_DIR'
                    sh 'cp index.html $BUILD_DIR/'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'echo "<testsuite><testcase classname=\"hello\" name=\"world\"/></testsuite>" > test-results.xml'
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
                input message: 'Approve deployment to IIS on port 9091?'
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'iis-creds', usernameVariable: 'IIS_USER', passwordVariable: 'IIS_PASS')]) {
                    bat '''
                    powershell -NoProfile -Command "
                        Import-Module WebAdministration;
                        $siteName = 'HelloWorldSite';
                        $physicalPath = env:WORKSPACE + '\\' + env:BUILD_DIR;
                        if (-not (Get-Website -Name $siteName -ErrorAction SilentlyContinue)) {
                            New-Website -Name $siteName -Port 9091 -PhysicalPath $physicalPath -Force;
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
                sh 'echo "Publishing to GitHub Packages using token $GITHUB_TOKEN"'
            }
        }
    }
    post {
        success {
            mail to: 'team@example.com', subject: 'Build and Deploy Successful', body: 'The deployment to IIS has finished.'
        }
    }
}
