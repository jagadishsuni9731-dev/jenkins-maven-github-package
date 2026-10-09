pipeline {
    agent {
        label 'linux-maven-agent'
    }
        options {
        skipDefaultCheckout(true)
    }
    environment {
        GITHUB_CREDS = credentials('Git-hub-package')
        JAVA_HOME    = tool 'jdk11'
        MAVEN_HOME   = tool 'maven3'
        PATH         = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {

stage('Checkout Code') {
    steps {
        script {
            checkout([
                $class: 'GitSCM',
                branches: [[name: '*/main']],
                doGenerateSubmoduleConfigurations: false,
                extensions: [],
                userRemoteConfigs: [[
                    url: 'https://github.com/jagadishsuni9731-dev/jenkins-maven-github-package.git'
                ]],
                gitTool: 'LinuxGit'
            ])
        }
    }
}

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "Running on EC2 Linux Agent"
                    hostname
                    whoami

                    echo "Java version:"
                    java -version

                    echo "Maven version:"
                    "$MAVEN_HOME/bin/mvn" -version
                '''
            }
        }

        stage('Build & Deploy') {
            steps {
                configFileProvider([
                    configFile(
                        fileId: 'maven-github-settings',
                        variable: 'MAVEN_SETTINGS'
                    )
                ]) {
                    sh '''
                        set -eu

                        export GH_USER="$GITHUB_CREDS_USR"
                        export GH_TOKEN="$GITHUB_CREDS_PSW"

                        echo "Building Maven project..."

                        "$MAVEN_HOME/bin/mvn" \
                            -s "$MAVEN_SETTINGS" \
                            -B clean package

                        echo "Deploying artifact to GitHub Packages..."

                        "$MAVEN_HOME/bin/mvn" \
                            -s "$MAVEN_SETTINGS" \
                            -B deploy
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment to GitHub Packages completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output for details.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
