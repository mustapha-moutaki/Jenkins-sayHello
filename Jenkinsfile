pipeline {
    agent any // tell the jenkins that any agent-node can excute this pipeline

    tools { //tools that we need to exicute this program -- important or failer
        jdk 'jdk17'
        maven 'Maven 3.9.0'
    }

    stages { // we customoze all stages that we wanna
        stage('Checkout') { // clone the project from the repsotory
        // it's important because it's the code source that we use
            steps {
                echo '🔍 Checking out code...'
                checkout scm // this is the responsible of getting the code from the repo
            }
        }

        stage('Build') { // build the project
            steps {
                echo '🔨 Building with Maven...'
                dir('demo') {      // we tell jenkes to go to demo directory t find pom.xml to run mvn
                    sh 'mvn clean package -DskipTests' // run mvn and skip tests
                }
            }
        }

        stage('Test') { // here we run the tests
            steps { // if the tests failed pipline stop - important--we can customize the failer behavior if needed
                echo '🧪 Running tests...'
                dir('demo') {
                    sh 'mvn test'
                }
            }
        }

        stage('Deploy') { // run the app in backgorund
            steps {
                echo '🚀 Starting Spring Boot app...'
                dir('demo') {
                    sh 'nohup java -jar target/*.jar > app.log 2>&1 &' // command to run the program in backgorund
                }
            }
        }
    }
}
