#!/usr/bin/env groovy

pipeline {
    agent {
        label 'built-in'
    }

    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        stage('check java') {
            steps {
                sh "java -version"
            }
        }

        stage('clean') {
            steps {
                sh "chmod +x mvnw"
                sh "./mvnw -ntp clean -P-webapp"
            }
        }
        stage('nohttp') {
            steps {
                sh "./mvnw -ntp checkstyle:check"
            }
        }

        stage('install tools') {
            steps {
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm@install-node-and-npm"
            }
        }

        stage('npm install') {
            steps {
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
            }
        }
        stage('backend tests') {
            steps {
                script {
                    try {
                        sh "./mvnw -ntp verify -P-webapp"
                    } catch(err) {
                        throw err
                    } finally {
                        junit '**/target/surefire-reports/TEST-*.xml,**/target/failsafe-reports/TEST-*.xml'
                    }
                }
            }
        }

        stage('frontend tests') {
            steps {
                script {
                    try {
                        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.npm.arguments='run test'"
                    } catch(err) {
                        throw err
                    } finally {
                        junit '**/target/test-results/TESTS-results-jest.xml'
                    }
                }
            }
        }

        stage('packaging') {
            steps {
                sh "./mvnw -ntp verify -P-webapp -Pprod -DskipTests"
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
    }
}