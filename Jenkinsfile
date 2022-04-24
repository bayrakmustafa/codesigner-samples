pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: "5"))
        disableConcurrentBuilds()
    }

    environment {
        ENV_FILE       = credentials('es-env-demo')
        GITHUB_TOKEN   = credentials('es-github-token')
        COMMAND        = 'sign'
    }

    stages {    
        stage('Prepare for Signing') {
            steps {
                sh "cp ${ENV_FILE} .env"
                sh "mkdir ${env.WORKSPACE}/artifacts"
            }
        }

        stage('Docker Pull Image') {
            steps {
                sh "echo ${GITHUB_TOKEN} | docker login ghcr.io -u bayrakmustafa --password-stdin"
                sh 'docker pull ghcr.io/bayrakmustafa/codesigner:latest'
            }
        }

        stage('Sign and Save Artifact') {
            steps {
                sh "docker run -i --rm --volume ${env.WORKSPACE}/artifacts:/codesign/output --env-file .env ghcr.io/bayrakmustafa/codesigner:latest ${COMMAND} -input_file_path=/codesign/examples/codesign.ps1 -output_dir_path=/codesign/output"
                sh "docker cp ${env.WORKSPACE}/artifacts/codesign.ps1 jenkins:${env.WORKSPACE}/codesign.ps1"
            }
            post {
                always {
                    archiveArtifacts artifacts: "${env.WORKSPACE}/artifacts/codesign.ps1", onlyIfSuccessful: true
                }
            }
        }
    
    }
}
