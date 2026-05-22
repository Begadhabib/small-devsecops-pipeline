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

        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false
                    }
                }
            }
        }

        stage('OWASP FS SCAN') {
    steps {
        dependencyCheck additionalArguments: '--scan ./Chess', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage('Docker Build') {
            steps {
                dir('Chess') {
                    sh '''
                        docker build -t my_chess_app .
                        docker tag my_chess_app begad1/my_chess_app:latest
                        docker images
                    '''
                }
            }
        }

        stage('Docker Scan Using Trivy') {
            steps {
                sh '''
                    set -e
                    trivy image \
                        --format json \
                        --output trivy-report.json \
                        begad1/my_chess_app:latest

                    echo "=== files generated ==="
                    ls -lah trivy-report.json
                '''
                archiveArtifacts artifacts: 'trivy-report.json'
            }
        }

        stage("Docker Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub-creds', toolName: 'docker') {
                        sh "docker push begad1/my_chess_app:latest"
                    }
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                sh '''
                    docker rm -f my_chess_app || true
                    docker run -d --name my_chess_app -p 3000:5000 begad1/my_chess_app:latest
                '''
            }
        }
        stage('Deploy to Kubernetes') {
    steps {
        script {
            withKubeConfig(credentialsId: 'k8s') {
                sh 'kubectl apply -f k8s/'
                sh 'kubectl get all -n chess'
            }
        }
    }
}
    }
}