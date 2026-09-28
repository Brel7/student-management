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
    }
    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            echo 'Pipeline réussi : le jar a été archivé.'
        }
        failure {
            echo 'Pipeline en échec : consulter la console et le Test Result.'
        }
    }
}
