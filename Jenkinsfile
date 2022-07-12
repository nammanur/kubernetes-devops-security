pipeline {
    agent any

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
            post {
                always {
                  junit 'target/surefire-reports/*.xml'
                  jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }
        stage('Docker Build and Push'){
            steps {
//                 withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                    sh 'printenv'
                    sh 'docker build -t nammanur/numeric-app:""$GIT_COMMIT"" .'
                    sh 'docker push nammanur/numeric-app:""$GIT_COMMIT""'
//                 }
            }
        }
    }
}