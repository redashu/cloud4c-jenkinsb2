# cloud4c-jenkinsb2

### understanding maven based java project build env

<img src="javae.png">

### installing jdk8 with maven in slave 1 only 

```
 yum install java-1.8.0-openjdk.x86_64  java-1.8.0-openjdk-devel.x86_64
yum install maven 
```

### creating jenkinsfile with maven build

```
pipeline {
    // planing to run this job on slave 1 only 
    agent {
        label 'slave1'
    }

    stages {
        stage('for cloning java project from github') {
            steps {
                echo 'we are cloning git repo'
                // using git internal keyword in pipeline
                git 'https://github.com/redashu/java-springboot.git'
                // verify it 
                sh 'ls'
            }
        }
        stage('using maven to build this in war file'){
            steps {
                echo 'please wait we are trying to build maven package'
                // using shell command 
                sh 'mvn clean package'
                // verify target folder
                sh 'ls target'
            }
        }
    }
    post {
        success {
            echo 'hey we did it'
        }
        failure {
            echo 'we can try again we still have way to go'
        }
    }
}

```
