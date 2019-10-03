#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    stage('clean') {
        sh "chmod +x mvnw"
        sh "./mvnw -ntp clean"
    }

    stage('install tools') {
        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm -DnodeVersion=v10.16.3 -DnpmVersion=6.11.3"
    }

    stage('npm install') {
        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
    }

    stage('backend tests') {
        try {
            sh "./mvnw -ntp verify"
        } catch(err) {
            throw err
        } finally {
            junit '**/target/test-results/**/TEST-*.xml'
        }
    }

    stage('frontend tests') {
        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.npm.arguments='run test'"
    }

    stage('build image and push to repo') {
	withCredentials([usernamePassword(credentialsId: 'region-harbor', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        	sh "./mvnw package -Pprod -DskipTests jib:build -Djib.to.image=harbor.teco.1-4.fi.teco.online/jdemo/app:latest -Djib.to.auth.username=$USERNAME -Djib.to.auth.password=$PASSWORD"
	}
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }
}
