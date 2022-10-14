pipeline {
    agent any

    stages {
        stage('validate') {
            steps {
                echo 'Code Validate'
                sh 'mvn compile'
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
