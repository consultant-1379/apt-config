#!/usr/bin/env groovy
pipeline {

    agent {
        node {
            label NODE_LABEL
        }
    }

    stages {
        stage('Lint JSON files') {
            steps {
                sh '''docker run --rm -v $(pwd):/data cytopia/jsonlint *.json'''
            }
        }
    }
}