pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: "5"))
        disableConcurrentBuilds()
    }

    tools {
        maven "MAVEN_3.8.5"
        jdk "JDK_9.0.4"
        gradle "GRADLE_7.4.2"
    }

    environment {
        ENV_FILE         = credentials('es-env-demo')
        USERNAME         = credentials('es-username')
        PASSWORD         = credentials('es-password')
        CREDENTIAL_ID    = credentials('es-crendential-id')
        TOTP_SECRET      = credentials('es-totp-secret')
        GITHUB_TOKEN     = credentials('es-github-token')
        ENVIRONMENT_NAME = 'TEST'
        COMMAND          = 'sign'

        PROJECT_NAME     = 'HelloWorld'
        PROJECT_VERSION  = '0.0.1'
        MAVEN_VERSION    = '3.8.5'
        JAVA_VERSION     = '9.0.4'
    }

    stages {    
        stage('Prepare for Signing') {
            steps {
                sh "cp ${ENV_FILE} .env"
                sh "mkdir ${env.WORKSPACE}/artifacts"
                sh "mkdir ${env.WORKSPACE}/packages"
            }
        }

        stage('Docker Pull Image') {
            steps {
                sh "echo ${GITHUB_TOKEN} | docker login ghcr.io -u bayrakmustafa --password-stdin"
                sh 'docker pull ghcr.io/bayrakmustafa/codesigner:latest'
            }
        }

        stage('Build Packages') {
            parallel {
                stage('Copy Powershell Script') {
                    steps {
                        sh "cp powershell/${env.PROJECT_NAME}.ps1 ${env.WORKSPACE}/packages/${env.PROJECT_NAME}.ps1"
                    }
                }
                stage('Build Maven') {
                    steps {
                        sh 'mvn clean install -f java/pom.xml'
                        sh "cp java/target/${env.PROJECT_NAME}-${env.PROJECT_VERSION}.jar ${env.WORKSPACE}/packages/${env.PROJECT_NAME}-Maven.jar"
                    }
                }
                stage('Build Gradle') {
                    steps {
                        sh 'gradle clean build -p java -PsetupType=jar'
                        sh "cp java/build/libs/${env.PROJECT_NAME}-${env.PROJECT_VERSION}.jar ${env.WORKSPACE}/packages/${env.PROJECT_NAME}-Gradle.jar"
                    }
                }
            }
        }

        stage('Sign and Save PS1 Artifact') {
            steps {
                sh "docker run -i --rm --dns 8.8.8.8 --network host --volume ${env.WORKSPACE}/packages:/codesign/examples --volume ${env.WORKSPACE}/artifacts:/codesign/output -e USERNAME=${USERNAME} -e PASSWORD=${password} -e CREDENTIAL_ID=${CREDENTIAL_ID} -e TOTP_SECRET=${TOTP_SECRET} -e ENVIRONMENT_NAME=${ENVIRONMENT_NAME} ghcr.io/bayrakmustafa/codesigner:latest ${COMMAND} -input_file_path=/codesign/examples/${env.PROJECT_NAME}.ps1 -output_dir_path=/codesign/output"
            }
            post {
                always {
                    archiveArtifacts artifacts: "artifacts/codesign.ps1", onlyIfSuccessful: true
                }
            }
        }

        stage('Sign and Save Maven JAR Artifact') {
            steps {
                sh "docker run -i --rm --dns 8.8.8.8 --network host --volume ${env.WORKSPACE}/packages:/codesign/examples --volume ${env.WORKSPACE}/artifacts:/codesign/output -e USERNAME=${USERNAME} -e PASSWORD=${password} -e CREDENTIAL_ID=${CREDENTIAL_ID} -e TOTP_SECRET=${TOTP_SECRET} -e ENVIRONMENT_NAME=${ENVIRONMENT_NAME} ghcr.io/bayrakmustafa/codesigner:latest ${COMMAND} -input_file_path=/codesign/examples/${env.PROJECT_NAME}-Maven.jar -output_dir_path=/codesign/output"
            }
            post {
                always {
                    archiveArtifacts artifacts: "artifacts/${env.PROJECT_NAME}-Maven.jar", onlyIfSuccessful: true
                }
            }
        }
    
        stage('Sign and Save Gradle JAR Artifact') {
            steps {
                sh "docker run -i --rm --dns 8.8.8.8 --network host --volume ${env.WORKSPACE}/packages:/codesign/examples --volume ${env.WORKSPACE}/artifacts:/codesign/output -e USERNAME=${USERNAME} -e PASSWORD=${password} -e CREDENTIAL_ID=${CREDENTIAL_ID} -e TOTP_SECRET=${TOTP_SECRET} -e ENVIRONMENT_NAME=${ENVIRONMENT_NAME} ghcr.io/bayrakmustafa/codesigner:latest ${COMMAND} -input_file_path=/codesign/examples/${env.PROJECT_NAME}-Gradle.jar -output_dir_path=/codesign/output"
            }
            post {
                always {
                    archiveArtifacts artifacts: "artifacts/${env.PROJECT_NAME}-Gradle.jar", onlyIfSuccessful: true
                }
            }
        }
    }
}
