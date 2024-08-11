pipeline {
    agent any
    environment {
        name = "Anmol"
        age = "23"
        platform = "Jenkins"
        uses = "Automation"
        IMAGE_NAME = 'anmolmishra334/jenkins_project'
        DOCKER_CREDENTIALS_ID = 'docker_cred'
        KUBERNETES_CONTEXT = 'minikube'
        IMAGE_TAG = "${env.BUILD_ID}"
        GIT_CREDENTIALS_ID = 'git-cred'
    }
    tools {
        jdk 'JDK'
        maven 'maven3'
    }
    stages {
        stage('Clear Workspace') {
            steps {
                cleanWs deleteDirs: true
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/anmolmishra334/Ekart.git'
            }
        }
 stage('Update Maven Version') {
            steps {
                script {
                    bat """
                        mvn versions:set -DnewVersion=1.0.${env.BUILD_ID}
                        mvn versions:commit
                    """
                }
            }
        }
        stage('Maven Clean Package Creation') {
            parallel {
                stage('Maven Validate') {
                    steps {
                        bat 'mvn validate'
                    }
                }
                stage('Maven Compile') {
                    steps {
                        bat 'mvn clean compile'
                    }
                }
                stage('Maven Test') {
                    steps {
                        bat 'mvn test'
                    }
                }
            }
        }
        stage('Maven Package Creation') {
            steps {
                bat 'mvn clean package'
            }
        }
        stage('Maven Archive Package') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            }
        } 
    stage('Upload Maven Package') {
    steps {
        script {
            bat "mvn deploy"
        }
    }
}
        stage('PR Approval Request') {
            steps {
                timeout(time: 300, unit: 'SECONDS') {
                    input id: 'YES', message: 'Enter Yes or No for PR Request Creation.', ok: 'YES', submitter: 'admin', submitterParameter: 'approval'
                }
            }
        }
        stage('Create Pull Request') {
            steps {
                script {
                    def branchName = "new_branch_${env.BUILD_ID}"
                    def repo = 'anmolmishra334/Petclinic'
                    def prTitle = 'Auto PR from Jenkins'
                    def prBody = 'This is an automated PR created by Jenkins after a successful build.'

                    withCredentials([string(credentialsId: "${GIT_CREDENTIALS_ID}", variable: 'GITHUB_TOKEN')]) {
                        bat """
                            curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" -H "Accept: application/vnd.github.v3+json" ^
                            https://api.github.com/repos/${repo}/pulls ^
                            -d "{\\"title\\": \\"${prTitle}\\", \\"body\\": \\"${prBody}\\", \\"head\\": \\"${branchName}\\", \\"base\\": \\"main\\"}"
                        """
                    }
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                bat 'copy /Y target\\*.war "C:\\tomcat\\webapps"'
            }
        }
        stage('Parameters and Environment Variables Usage') {
            parallel {
                stage('Choice Parameter') {
                    steps {
                        echo "${params.choices}"
                    }
                }
                stage('Single Parameter Input') {
                    steps {
                        echo "${params.string_parameter_input}"
                    }
                }
                stage('Multi-line Parameter Input') {
                    steps {
                        echo "${params.multi_line_string_parameter}"
                    }
                }
                stage('Environment Variables in Jenkinsfile') {
                    steps {
                        echo "My name is ${name} and my age is ${age}, I am using ${platform} platform for ${uses}."
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }
        stage('Login to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        echo 'Logged in to Docker Hub'
                    }
                }
            }
        }
        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image("${IMAGE_NAME}:${env.BUILD_ID}").push()
                        echo 'Image pushed to Docker Hub'
                    }
                }
            }
        }
       stage('Deploy Docker Container') {
            steps {
                script {
                    // Stop any existing container with the same name
                    bat "docker rm -f ${env.BUILD_ID}_container || true"
                    
                    // Run the Docker container on port 8082
                    bat "docker run -d -p 8083:8080 --name ${env.BUILD_ID}_container ${IMAGE_NAME}:${env.BUILD_ID}"
                    echo "Container deployed on port 8082"
                }
            }
        }
    }
}
