pipeline {
    agent any
    stages {
        stage('Build Artifact') {
            steps {
                sh "mvn clean package -DskipTests=true"
                archive 'target/*.jar'
            }
        }

        stage('UNIT test & jacoco') {
            steps {
                sh "mvn test"
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }

        stage('Mutation Tests - PIT') {
            steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }

            post {
                always {
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([string(credentialsId: 'password-dockerHub', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh 'sudo docker login -u yantou -p $DOCKER_HUB_PASSWORD'
                    sh 'printenv'
                    sh "sudo docker build -t yantou/devops-app:$GIT_COMMIT ."
                    sh "sudo docker push yantou/devops-app:$GIT_COMMIT"
                }
            }
        }

        stage('Deployment Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'config']) {
                    sh "sed -i 's#replace#yantou/devops-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        }
    }
}
