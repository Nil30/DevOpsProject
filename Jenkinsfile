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
    }
}
