// pipeline {
//     agent any
//     stages{
//         stage ("build") {
//             steps{
//                 echo "hi"
//             }
//         }
//         stage ("deploy") {
//             steps{
//                 echo "hello"
//             }
//         }
//     }
// }

pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        packageVersion = ''
        nexusURL = '172.31.83.26:8081'
    }
    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }
    // parameters {
    //     string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

    //     text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

    //     booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

    //     choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

    //     password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    // }
    //build
    stages {
        stage('Get the version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    packageVersion = packageJson.version
                    echo "application version: $packageVersion"
                }
            }
        }
        stage('install dependencies') {
            steps {
               sh """
                   npm install
               """
            }
        }
        stage('Build') {
            steps {
                sh """
                    ls -la
                    zip -q -r catalogue.zip ./* -x ".git" -x ".zip"
                    ls -ltr
                """
            }
        }
        stage('Public artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusURL}",
                    groupId: 'com.roboshop',
                    version: "${packageVersion}",
                    repository: 'catalogue',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [artifactId: 'catalogue',
                        classifier: '',
                        file: 'catalogue.zip',
                        type: 'zip']
                    ]
                )
            }
        }
        stage('Deploy') {
            steps {
                script {
                    def params = [
                        string(name: 'version', value: "${packageVersion}"),
                        string(name: 'environment', value: "dev")
                    ]
                    build job: "catalogue-dep", wait: true, parameters: params
                }
            }
        }
    //post build
    post {
        always {
            echo 'I will always say HELLO again'
            deleteDir()
        }
        failure {
            echo 'This run when pipeline failed'
        }
        success {
            echo 'This pipeline is sucess'
        }
    }
}

