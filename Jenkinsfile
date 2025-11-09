pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'Maven 3.9.0'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '🔍 Checking out code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Building with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running tests...'
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Starting Spring Boot app...'
                sh 'nohup java -jar target/*.jar > app.log 2>&1 &'
            }
        }
    }
}