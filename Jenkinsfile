pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout from Git') {
            steps {
                git branch: 'feature/CI', 
                    url: 'https://github.com/Begadhabib/small-devsecops-pipeline.git'
            }
        }

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Game \
                        -Dsonar.projectKey=Game
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    ls -la
                    if [ -f package.json ]; then
                        npm install
                    else
                        echo "No package.json found - skipping npm stage"
                    fi
                '''
            }
        }
    }
}