pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'mysonar'
    }

    stages {
        stage("Clean") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout Code") {
            steps {
                git branch: 'main', url: 'https://github.com/Vaasanth3/Zomato-Repo.git'     
            }
        }


        stage("Code Quality Analysis") {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=zomato \
                        -Dsonar.projectKey=zomato
                    '''
                }
            }
        }

        // Remove this stage if pipeline fails due to Quality Gates
        stage("Quality Gates") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        // Remove this stage if pipeline fails due to OWASP
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DOCKER-HUB') {
                        sh 'docker build -t vaasanth/myzomato:v1 .'
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh 'trivy fs . > trivyfs.txt'
                sh 'trivy image vaasanth/myzomato:v1'
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withDockerRegistry(credentialsId: 'DOCKER-HUB', url: 'https://index.docker.io/v1/') {
                    sh 'docker push vaasanth/myzomato:v1'
                }
            }
        }

        stage("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 vaasanth/myzomato:v1'
            }
        }
    }
}
