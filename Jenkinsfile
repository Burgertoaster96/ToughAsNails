@Library('forge-shared-library')_

pipeline {
    options {
        disableConcurrentBuilds()
    }
    agent {
        docker {
            image 'gradle:7-jdk16'
        }
    }
    environment {
        GRADLE_ARGS = '-Dorg.gradle.daemon.idletimeout=5000'
    }
    stages {
        stage('fetch') {
            steps {
                checkout scm
            }
        }
        stage('setup') {
            steps {
                withGradle {
                    sh './gradlew ${GRADLE_ARGS} --refresh-dependencies'
                }
                script {
                    env.MYVERSION = sh(returnStdout: true, script: './gradlew :properties -q | grep "^version:" | awk \'{print $2}\'').trim()
                }
            }
        }
        stage('changelog') {
            when {
                not {
                    changeRequest()
                }
            }
            steps {
                writeChangelog(currentBuild, "build/ToughAsNails-${env.MYVERSION}-changelog.txt")
            }
        }
        stage('publish') {
            when {
                not {
                    changeRequest()
                }
            }
            environment {
                CURSE_API_KEY = credentials('curse-api-key')
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'maven-adubbz-user', usernameVariable: 'MAVEN_USER', passwordVariable: 'MAVEN_PASSWORD')]) {
                    sh './gradlew ${GRADLE_ARGS} publish'
                }
                withGradle {
                    sh './gradlew ${GRADLE_ARGS} curseforge -PcurseApiKey=${CURSE_API_KEY}'
                }
            }
        }
    }
}