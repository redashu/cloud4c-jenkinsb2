# cloud4c-jenkinsb2

### Rev 1

<img src="rev1.png">

### revision 2

<img src="rev2.png">

### user and notification service 

<img src="ut.png">

### RBAC 

<img src="rbac.png">

## Notification & alert 

### SMTP 

<img src="smtp.png">

### jenkisfile job

```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
        stage('testing docker connection'){
            steps {
                echo 'testing connection of docker'
                sh 'docker version'
                sleep 3 
                // i want to send an email notication about docker successfull connection
                mail bcc: '', body: '''yeahhhh ohhhhhhhhhhh!!
we got docker connection established hurrrahh''', cc: 'navinyab95@gmail.com', from: '', replyTo: '', subject: 'docker testing status', to: 'ashutoshh@delvex.io'
                
            }
        }
    }
}


```

