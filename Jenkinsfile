pipeline {
    agent any

    stages {
        stage('validate') {
            steps {
                echo 'Code Validate'
                sh 'mvn complie'
            }
        }
        stage('UnitTest') {
            steps {
                echo 'Unit Testing'
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Sonar Analysing'
                sh 'mvn sonar:sonar -Dsonar.host.url=http://3.111.245.205:9000 -Dsonar.login=b7d69f5f2c5351a6f9f84c3c527349510f34d190'
            }
        }
        stage('Public Artifact') {
            steps {
                echo 'Providing Artifact'
                sh 'mvn deploy'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying App on Tomcat Server'
                sh 'cd /opt/apache-tomcat-7.0.94/webapps/'
                sh 'sudo wget --user admin --password admin123 http://3.111.245.205:8081/nexus/service/local/repositories/releases/content/com/web/cal/WebAppCal/1.3.8/WebAppCal-1.3.8.war'
            }
        }
    }
}
