pipeline {
    agent any
    environment {
        deploymentName = "devsecops"
        containerName = "devsecops-container"
        serviceName = "devsecops-svc"
        imageName = "nammanur/numeric-app:${GIT_COMMIT}"
        applicationURL = "http://devsecops-mydemo.centralindia.cloudapp.azure.com:30516/"
        applicationURI = "/increment/99"
    }
    stages {
        stage('Build Artifact') {
            steps {
                sh "mvn clean package -DskipTests=true"
                archive 'target/*.jar'
            }
        }
        stage('Unit Tests-Junit and JaCoCo') {
            steps {
                sh "mvn test"
            }

        }
        stage('Mutation Tests - PIT') {
          steps {
            sh "mvn org.pitest:pitest-maven:mutationCoverage"
          }
        }
        stage('SonarQube - SAST') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application1 -Dsonar.host.url=http://devsecops-mydemo.centralindia.cloudapp.azure.com:9000 -Dsonar.login=b27079f4478df3a568cdf73f557084fa7f10eb90"
                  }
                  timeout(time: 2, unit: 'MINUTES') {
                     script {
                        waitForQualityGate abortPipeline: true
                    }
                 }
              }
        }
//         stage('Vulnerability Scan - Docker ') {
//             steps {
//                 sh "mvn dependency-check:check"
//             }
//             post {
//                 always {
//                     dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
//                 }
//             }
//         }
        stage('Vulnerability Scan - Docker') {
            steps {
                parallel(
                    "Dependency Scan": {
                    sh "mvn dependency-check:check"
                },
                "Trivy Scan": {
                    sh "bash trivy-docker-image-scan.sh"
                }
                )
            }
        }
        stage('Docker Build and Push'){
            steps {
                withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                    sh 'printenv'
                    sh 'sudo docker build -t nammanur/numeric-app:""$GIT_COMMIT"" .'
                    sh 'docker push nammanur/numeric-app:""$GIT_COMMIT""'
                }
            }
        }
        stage('K8S Deployment - DEV') {
            steps {
                parallel(
                    "Deployment": {
                        withKubeConfig([credentialsId: 'kubeconfig']) {
                            sh "bash k8s-deployment.sh"
                        }
                    },
                    "Rollout Status": {
                        withKubeConfig([credentialsId: 'kubeconfig']) {
                            sh "bash k8s-deployment-rollout-status.sh"
                        }
                    }
                )
            }
        }
    }
    post {
        always {
            junit 'target/surefire-reports/*.xml'
            jacoco execPattern: 'target/jacoco.exec'
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
//             success {
//             }
//             failure {
//             }
    }

}