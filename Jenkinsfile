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

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'feature/CI', 
                    url: 'https://github.com/Begadhabib/small-devsecops-pipeline.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    if [ -f Chess/package.json ]; then
                        cd Chess && npm install
                    else
                        echo "No package.json found - skipping npm stage"
                    fi
                '''
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
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
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
        stage('Docker Build') {
            steps {
                sh '''
                    cd Chess
                    docker build -t my_chess_app .
                    docker images
                '''
            }
        }
        stage('Docker Scan Using Trivy') {
    steps {
        sh '''
            set -e

            trivy image \
            --format json \s
            --output trivy-report.json \
            my_chess_app:latest

            echo "=== files generated ==="
            ls -lah trivy-report.json
        '''

        archiveArtifacts artifacts: 'trivy-report.json'
    }
}
stage('Deploy to container'){
            steps{
                sh 'docker run -d --name my_chess_app -p 3000:5000 my_chess_app:latest'
            }
        }
    }
}