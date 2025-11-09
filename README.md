![Jenkins Logo](https://www.jenkins.io/images/logos/jenkins/jenkins.png)

# 🚀 Jenkins Pipeline Guide for Spring Boot Projects

> **"Automate like a boss, deploy like a pro, and sleep like a baby!"**

Welcome to your ultimate Jenkins companion guide! This documentation will transform you from a Jenkins newbie to a CI/CD wizard. Whether you're setting up your first pipeline or your hundredth, this guide has got your back.

---

## 📋 Table of Contents

- [What is Jenkins Anyway?](#what-is-jenkins-anyway)
- [Before You Start - The Checklist](#before-you-start---the-checklist)
- [Project Structure That Makes Sense](#project-structure-that-makes-sense)
- [Setting Up Your First Pipeline](#setting-up-your-first-pipeline)
- [The Jenkinsfile - Your Automation Recipe](#the-jenkinsfile---your-automation-recipe)
- [Stage by Stage Breakdown](#stage-by-stage-breakdown)
- [Common Pitfalls & How to Dodge Them](#common-pitfalls--how-to-dodge-them)
- [Pro Tips & Tricks](#pro-tips--tricks)
- [What's Next?](#whats-next)

---

## 🤔 What is Jenkins Anyway?

Think of Jenkins as your tireless robot assistant that:

- ✓ Builds your code automatically when you push changes
- ✓ Runs all your tests so you don't have to
- ✓ Deploys your application without breaking a sweat
- ✓ Notifies you if something goes wrong
- ✓ Never complains, never sleeps, never asks for coffee

**In simple terms:** Jenkins takes the boring, repetitive stuff off your plate so you can focus on writing awesome code.

**Key Benefits:**
- **Save Time:** No more manual builds and deployments
- **Catch Bugs Early:** Tests run automatically on every change
- **Deploy Confidently:** Consistent, repeatable deployment process
- **Sleep Better:** Know that your CI/CD pipeline is watching over your code

---

## ✅ Before You Start - The Checklist

Make sure you have these tools ready:

| Tool | Why You Need It | How to Check |
|------|-----------------|--------------|
| **Jenkins** | The CI/CD engine | Visit `http://localhost:8080` |
| **Java JDK** | To compile Spring Boot apps | `java -version` |
| **Maven** | To build your project | `mvn -version` |
| **Git** | To pull code from repositories | `git --version` |
| **GitHub Account** | To host your code | Your repo URL |

**Pro Tip:** Make sure your Java version matches what your Spring Boot project needs. Spring Boot 3.x needs Java 17 or higher!

---

## 📁 Project Structure That Makes Sense

Here's what a well-organized Spring Boot project looks like:

```
my-spring-boot-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── demo/
│   │   │               └── DemoApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
│           └── com/
│               └── example/
│                   └── demo/
│                       └── DemoApplicationTests.java
├── Jenkinsfile              ← Your automation script lives here
├── pom.xml                  ← Maven configuration
├── README.md                ← Project documentation
└── .gitignore
```

**Important Notes:**

- → The `Jenkinsfile` should be at the root of your repository
- → The `pom.xml` (Maven) or `build.gradle` (Gradle) must be in the same directory
- → If your Spring Boot app is in a subfolder, you'll need to navigate there in your Jenkinsfile using `dir('subfolder-name')`

---

## 🎯 Setting Up Your First Pipeline

### Step 1: Create a New Pipeline Job

1. Open Jenkins dashboard (`http://localhost:8080`)
2. Click **"New Item"** in the left sidebar
3. Enter a name like `spring-boot-pipeline`
4. Select **"Pipeline"**
5. Click **OK**

### Step 2: Configure Source Code Management

In the Pipeline configuration page:

1. Scroll to **"Pipeline"** section
2. Set **Definition** to: `Pipeline script from SCM`
3. Set **SCM** to: `Git`
4. Enter your **Repository URL**: `https://github.com/username/your-repo.git`
5. Set **Branch Specifier** to: `*/main` (or your default branch)
6. Set **Script Path** to: `Jenkinsfile`
7. Click **Save**

### Step 3: Configure Java & Maven Tools

Before running pipelines, tell Jenkins where to find Java and Maven:

1. Go to **Manage Jenkins** → **Global Tool Configuration**
2. Scroll to **JDK** section:
    - Click **Add JDK**
    - Name it: `jdk17` (or whatever version you're using)
    - Uncheck "Install automatically" if you already have it installed
    - Or select the version to auto-install
3. Scroll to **Maven** section:
    - Click **Add Maven**
    - Name it: `Maven 3.9.11` (match your Maven version)
    - Select the version or provide the installation path
4. Click **Save**

**Critical:** The names you use here (`jdk17`, `Maven 3.9.11`) must match exactly in your Jenkinsfile!

---

## 📝 The Jenkinsfile - Your Automation Recipe

Here's a complete, production-ready Jenkinsfile for a Spring Boot project:

```groovy
pipeline {
    agent any

    tools {
        jdk 'jdk17'              // Must match the name in Jenkins Global Tools
        maven 'Maven 3.9.11'     // Must match the name in Jenkins Global Tools
    }

    environment {
        APP_NAME = 'spring-boot-demo'
        DEPLOY_PORT = '8080'
    }

    stages {
        
        stage('📥 Checkout') {
            steps {
                echo '================================================'
                echo '    Pulling latest code from repository...'
                echo '================================================'
                checkout scm
            }
        }

        stage('🏗️ Build') {
            steps {
                echo '================================================'
                echo '    Building the project with Maven...'
                echo '================================================'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('🧪 Test') {
            steps {
                echo '================================================'
                echo '    Running unit and integration tests...'
                echo '================================================'
                sh 'mvn test'
            }
        }

        stage('📦 Archive Artifacts') {
            steps {
                echo '================================================'
                echo '    Archiving the JAR file...'
                echo '================================================'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('🚀 Deploy') {
            steps {
                echo '================================================'
                echo '    Deploying Spring Boot application...'
                echo '================================================'
                sh '''
                    # Kill any existing instance
                    pkill -f ${APP_NAME} || true
                    
                    # Start the application
                    nohup java -jar target/*.jar > app.log 2>&1 &
                    
                    # Wait a bit and check if it's running
                    sleep 5
                    echo "Application deployed successfully!"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above.'
        }
        always {
            echo '🧹 Cleaning up workspace...'
            cleanWs()
        }
    }
}
```

---

## 🔍 Stage by Stage Breakdown

Let's dissect each part of the Jenkinsfile:

### The Pipeline Structure

```groovy
pipeline {
    agent any
    // ... rest of the pipeline
}
```

- **`pipeline {}`**: Declares this is a declarative pipeline (easier to read and write)
- **`agent any`**: Run this pipeline on any available Jenkins agent/node

### Tools Declaration

```groovy
tools {
    jdk 'jdk17'
    maven 'Maven 3.9.11'
}
```

- Tells Jenkins which JDK and Maven to use
- Names must match **exactly** what you configured in Global Tool Configuration
- Jenkins will automatically set up PATH variables for these tools

### Environment Variables

```groovy
environment {
    APP_NAME = 'spring-boot-demo'
    DEPLOY_PORT = '8080'
}
```

- Define variables you can use throughout the pipeline
- Access them with `${APP_NAME}` syntax
- Great for configuration that might change

### Stage 1: Checkout

```groovy
stage('📥 Checkout') {
    steps {
        checkout scm
    }
}
```

- **`checkout scm`**: Special command that pulls code from your configured Git repository
- **SCM** = Source Code Management (Git, in our case)
- Jenkins automatically knows which branch and repo to use from your job configuration

### Stage 2: Build

```groovy
stage('🏗️ Build') {
    steps {
        sh 'mvn clean package -DskipTests'
    }
}
```

- **`mvn clean`**: Removes old build files
- **`package`**: Compiles code and creates JAR file
- **`-DskipTests`**: Skips running tests (we'll run them separately in the next stage)
- **`sh`**: Runs a shell command

**If your project is in a subfolder:**
```groovy
stage('🏗️ Build') {
    steps {
        dir('demo') {  // Navigate to subfolder first
            sh 'mvn clean package -DskipTests'
        }
    }
}
```

### Stage 3: Test

```groovy
stage('🧪 Test') {
    steps {
        sh 'mvn test'
    }
}
```

- Runs all JUnit tests in your project
- If any test fails, the pipeline stops here
- Test reports are automatically collected by Jenkins

### Stage 4: Archive Artifacts

```groovy
stage('📦 Archive Artifacts') {
    steps {
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }
}
```

- Saves the built JAR file in Jenkins
- You can download it later from the Jenkins UI
- **`fingerprint: true`**: Tracks which builds produced which artifacts

### Stage 5: Deploy

```groovy
stage('🚀 Deploy') {
    steps {
        sh '''
            pkill -f ${APP_NAME} || true
            nohup java -jar target/*.jar > app.log 2>&1 &
            sleep 5
        '''
    }
}
```

- **`pkill -f ${APP_NAME}`**: Kills any running instance of your app
- **`|| true`**: Don't fail if nothing was running
- **`nohup`**: Keeps the app running even after Jenkins disconnects
- **`> app.log 2>&1`**: Redirects output and errors to a log file
- **`&`**: Runs in the background
- **`sleep 5`**: Waits to ensure the app starts properly

### Post Actions

```groovy
post {
    success {
        echo '✅ Pipeline completed successfully!'
    }
    failure {
        echo '❌ Pipeline failed.'
    }
    always {
        cleanWs()
    }
}
```

- **`post`**: Runs after all stages complete
- **`success`**: Only runs if everything passed
- **`failure`**: Only runs if something failed
- **`always`**: Runs no matter what
- **`cleanWs()`**: Cleans up the workspace to save disk space

---

## ⚠️ Common Pitfalls & How to Dodge Them

### Problem 1: "Tool not found" Error

**Error message:**
```
/bin/sh: mvn: command not found
```

**Solution:**
- Make sure Maven is configured in Jenkins Global Tool Configuration
- The name in your Jenkinsfile must match exactly
- Restart Jenkins after adding tools

### Problem 2: "pom.xml not found"

**Error message:**
```
The goal you specified requires a project to execute but there is no POM in this directory
```

**Solution:**
- Your `pom.xml` might be in a subfolder
- Use `dir('subfolder-name')` to navigate:
```groovy
stage('Build') {
    steps {
        dir('demo') {
            sh 'mvn clean package'
        }
    }
}
```

### Problem 3: Java Version Mismatch

**Error message:**
```
Unsupported class file major version 61
```

**Solution:**
- Your code was compiled with a newer Java version
- Update the JDK in Jenkins tools configuration
- Or downgrade your Spring Boot project's Java version

### Problem 4: Port Already in Use

**Error message:**
```
Web server failed to start. Port 8080 was already in use.
```

**Solution:**
- Kill the old process before deploying:
```groovy
sh 'pkill -f spring-boot || true'
sh 'nohup java -jar target/*.jar &'
```

### Problem 5: Permission Denied

**Error message:**
```
Permission denied: Cannot execute java
```

**Solution:**
- Jenkins user doesn't have execute permissions
- Run: `chmod +x /path/to/java`
- Or run Jenkins with proper user permissions

---

## 💡 Pro Tips & Tricks

### Tip 1: Use Parameters for Flexibility

Make your pipeline configurable:

```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deploy to which environment?')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    }
    
    stages {
        stage('Test') {
            when {
                expression { params.RUN_TESTS }
            }
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

### Tip 2: Add Email Notifications

Get notified when builds fail:

```groovy
post {
    failure {
        mail to: 'your-email@example.com',
             subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
             body: "Check console output at ${env.BUILD_URL}"
    }
}
```

### Tip 3: Use Build Triggers

Auto-build on Git push:

1. In your Jenkins job, check **"GitHub hook trigger for GITScm polling"**
2. In GitHub, go to repo → Settings → Webhooks
3. Add webhook: `http://your-jenkins-url/github-webhook/`
4. Now every `git push` triggers a build!

### Tip 4: Health Checks After Deployment

Make sure your app actually started:

```groovy
stage('Deploy') {
    steps {
        sh 'nohup java -jar target/*.jar &'
        sh 'sleep 10'
        sh 'curl -f http://localhost:8080/actuator/health || exit 1'
    }
}
```

### Tip 5: Use Credentials Safely

Never hardcode passwords! Use Jenkins credentials:

```groovy
environment {
    DB_PASSWORD = credentials('database-password')
}

stages {
    stage('Deploy') {
        steps {
            sh 'java -jar app.jar --spring.datasource.password=${DB_PASSWORD}'
        }
    }
}
```

### Tip 6: Parallel Stages for Speed

Run tests in parallel:

```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Tests') {
            steps {
                sh 'mvn verify'
            }
        }
    }
}
```

---

## 🎓 What's Next?

Now that you've mastered Jenkins basics, here's what to explore next:

### Level Up Your Pipeline

- **→ Docker Integration:** Build and push Docker images
  ```groovy
  sh 'docker build -t my-app:${BUILD_NUMBER} .'
  sh 'docker push my-app:${BUILD_NUMBER}'
  ```

- **→ Kubernetes Deployment:** Deploy to K8s clusters
  ```groovy
  sh 'kubectl apply -f deployment.yaml'
  ```

- **→ SonarQube Integration:** Add code quality checks
  ```groovy
  sh 'mvn sonar:sonar'
  ```

### Advanced Jenkins Features

- **→ Multibranch Pipelines:** Automatic pipelines for each Git branch
- **→ Shared Libraries:** Reuse pipeline code across projects
- **→ Blue Ocean UI:** Modern, visual pipeline interface
- **→ Jenkins Configuration as Code (JCasC):** Version control your Jenkins setup

### Monitoring & Observability

- **→ Prometheus + Grafana:** Monitor Jenkins performance
- **→ ELK Stack:** Aggregate and analyze build logs
- **→ Slack/Teams Integration:** Team notifications

---

## 📚 Quick Reference Commands

### Maven Commands

| Command | What it Does |
|---------|-------------|
| `mvn clean` | Delete old build files |
| `mvn compile` | Compile source code |
| `mvn test` | Run unit tests |
| `mvn package` | Create JAR/WAR file |
| `mvn install` | Install to local Maven repo |
| `mvn verify` | Run integration tests |

### Jenkins CLI Tricks

```bash
# Trigger a build remotely
curl -X POST http://localhost:8080/job/my-job/build

# Get build status
curl http://localhost:8080/job/my-job/lastBuild/api/json

# Download artifact
curl -O http://localhost:8080/job/my-job/lastSuccessfulBuild/artifact/target/app.jar
```

### Useful Pipeline Syntax

```groovy
// Current build number
env.BUILD_NUMBER

// Current Git branch
env.GIT_BRANCH

// Workspace directory
env.WORKSPACE

// Timeout for a stage
timeout(time: 10, unit: 'MINUTES') {
    sh 'mvn test'
}

// Retry on failure
retry(3) {
    sh 'mvn deploy'
}
```

---

## 🎉 Final Words

Congratulations! You now have everything you need to create, understand, and maintain Jenkins pipelines for Spring Boot projects. Remember:

- ✓ Start simple, then add complexity as needed
- ✓ Always test your pipeline changes in a development environment first
- ✓ Use meaningful stage names and echo statements for debugging
- ✓ Version control your Jenkinsfile just like your code
- ✓ Don't be afraid to experiment - you can always rollback!

**"The best time to automate was yesterday. The second best time is now."**

Happy building, testing, and deploying! 🚀

---

## 📞 Need Help?

- **Jenkins Documentation:** [jenkins.io/doc](https://jenkins.io/doc)
- **Spring Boot Guides:** [spring.io/guides](https://spring.io/guides)
- **Stack Overflow:** Tag your questions with `jenkins` and `spring-boot`

---

*Created with ♥ for developers who believe in automation*

*Version 1.0 | Last Updated: November 2025*