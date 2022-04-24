pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: "5"))
        disableConcurrentBuilds()
    }

    environment {
        ENV_FILE         = credentials('es-env-demo')
        USERNAME         = credentials('es-username')
        password         = credentials('es-password')
        CREDENTIAL_ID    = credentials('es-credantial-id')
        TOTP_SECRET      = credentials('es-totp-secret')
        GITHUB_TOKEN     = credentials('es-github-token')
        ENVIRONMENT_NAME = 'TEST'
        COMMAND          = 'sign'
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
                sh "docker run -i --rm --dns 8.8.8.8 --network host --volume ${env.WORKSPACE}/artifacts:/codesign/output 
                    -e USERNAME=${USERNAME} -e PASSWORD=${password} -e CREDENTIAL_ID=${CREDENTIAL_ID} -e TOTP_SECRET=${TOTP_SECRET} 
                    ghcr.io/bayrakmustafa/codesigner:latest ${COMMAND} 
                    -input_file_path=/codesign/examples/codesign.ps1 -output_dir_path=/codesign/output"
            }
            post {
                always {
                    archiveArtifacts artifacts: "artifacts/codesign.ps1", onlyIfSuccessful: true
                }
            }
        }
    
    }
}
