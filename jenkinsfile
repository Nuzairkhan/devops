pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'uname'
            }
        }
        stage('Test') {
            steps {
                sh 'cat /etc/os-release'
            }
        }
        stage('Deploy') {
            steps {
                sh 'cat /proc/cpuinfo'
            }
        }
    }
}
