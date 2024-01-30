pipeline {
    agent {
        node{
            label 'maven'
        }
    }

    stages {
        stage('hello') {
            steps {
                echo 'Hello World'
            }
        }
         stage('clone-git') {
            steps {
                git branch: 'main', url: 'https://github.com/MSathishD/tweet-trend-new.git'
            }
        }
    }
}