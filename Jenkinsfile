def envProps

pipeline {
    
    agent none

    environment {
        MAVEN_IMAGE = 'maven:3.6.2-jdk-11'
        MAVEN_IMAGE_ARGS = '-v /root/jenkins_home/ssh:/ssh -v /root/jenkins_home/.m2:/root/.m2:z -u root'
        NODE_IMAGE = 'node:12.13.0'
        NODE_IMAGE_ARGS = '-u 0:0'
        DOCKER_IMAGE = ''
    }

    stages {

        stage('Read .env') {
            
            agent any

            steps{
                echo 'Read .env'
                echo '******************************'
                script {
                    def envFile = readFile("${env.JENKINS_HOME}/deploy/account.env");
                    def map = [:], lines = envFile.split("\r?\n"); 
                    for (def line : lines) {
                        def arr = line.split("=");
                        map.put(arr[0], arr[1]);
                    }
                    envProps = map;
                }
            }
        }

        stage('Yarn Install') {

            agent {
                docker {
                    image "${env.NODE_IMAGE}"
                    args "${env.NODE_IMAGE_ARGS}"
                }
            }

            steps {
                echo 'Yarn Install'
                echo '******************************'
                sh 'node --version'
                sh 'yarn --version'
                sh 'cd frontend && yarn install'
            }
        }

        stage('Yarn Build') {

            agent {
                docker {
                    image "${env.NODE_IMAGE}"
                    args "${env.NODE_IMAGE_ARGS}"
                }
            }

            steps {
                echo 'Yarn Build'
                echo '******************************'
                sh 'cd frontend && yarn build'
            }
        }
        
        stage('Mvn Install') {
        
            agent {
                docker {
                    image "${env.MAVEN_IMAGE}" 
                    args "${env.MAVEN_IMAGE_ARGS}"
                    reuseNode true
                }
            }

            steps {
                echo 'Mvn Install'
                echo '******************************'
                sh 'java -version'
                sh 'mvn -version'
                sh 'mvn -U clean install -DskipTests=true'
            }
        }

        stage('Mvn Test') {
        
            agent {
                docker {
                    image "${env.MAVEN_IMAGE}" 
                    args "${env.MAVEN_IMAGE_ARGS}"
                    reuseNode true
                }
            }

            steps {
                echo 'Mvn Test'
                echo '******************************'
                sh 'mvn test'
            }
        }

        stage('Docker Build') {

            agent any

            steps{
                echo 'Docker Build'
                echo '******************************'
                script {
                  DOCKER_IMAGE = docker.build envProps.REGISTRY_NAME + ":latest"
                }
            }
        }

        stage('Docker Push') {

            agent any

            steps{
                echo 'Docker Push'
                echo '******************************'
                script {
                    docker.withRegistry(envProps.REGISTRY_URL, envProps.REGISTRY_CREDENTIAL) {
                        DOCKER_IMAGE.push()
                    }
               }
            }
        }

        stage('Deploy') {

            agent any

            steps{
                echo 'Deploy'
                echo '******************************'
                sh "sh ${env.JENKINS_HOME}/deploy/${envProps.DEPLOY_SH_FILE}"
            }
        }

    }
}
