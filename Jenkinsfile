#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    gitlabCommitStatus('build') {
        docker.image('jhipster/jhipster:v6.9.1').inside('-u jhipster -e MAVEN_OPTS="-Duser.home=./"') {
            stage('check java') {
                sh "java -version"
            }

            stage('clean') {
                sh "chmod +x mvnw"
                sh "./mvnw -ntp clean -P-webpack"
            }
            stage('nohttp') {
                sh "./mvnw -ntp checkstyle:check"
            }

            stage('install tools') {
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm -DnodeVersion=v12.16.1 -DnpmVersion=6.14.5"
            }

            stage('npm install') {
                sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
            }

            stage('backend tests') {
                try {
                    sh "./mvnw -ntp verify -P-webpack"
                } catch(err) {
                    throw err
                } finally {
                    junit '**/target/test-results/**/TEST-*.xml'
                }
            }

            stage('frontend tests') {
                try {
                   npm install
                   npm test
                } catch(err) {
                    throw err
                } finally {
                    junit '**/target/test-results/**/TEST-*.xml'
                }
            }

            stage('packaging') {
                sh "./mvnw -ntp verify -P-webpack deploy -Pprod -DskipTests"
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
            stage('quality analysis') {
                withSonarQubeEnv('sonar') {
                    sh "./mvnw -ntp initialize sonar:sonar"
                }
            }
        }

        def dockerImage
        stage('publish docker') {
            // A pre-requisite to this step is to setup authentication to the docker registry
            // https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#authentication-methods
            sh "./mvnw -ntp jib:build"
        }
    }
}
