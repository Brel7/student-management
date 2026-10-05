pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Commit') {
            steps {
                git branch: 'main', url: 'https://github.com/Brel7/student-management.git'
                sh 'git log -1 --pretty=format:"%h | %an | %s"'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test unitaire') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t brel7/student-management:${BUILD_NUMBER} \
                        -t brel7/student-management:latest \
                        .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            -u "$DOCKERHUB_USERNAME" \
                            --password-stdin

                        docker push brel7/student-management:${BUILD_NUMBER}
                        docker push brel7/student-management:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            echo 'Pipeline réussi : le jar a été archivé et l’image Docker a été publiée.'
        }

        failure {
            echo 'Pipeline en échec : consulter la console et le Test Result.'
        }
    }
}
