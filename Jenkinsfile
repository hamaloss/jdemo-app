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

    stage('build image') {
       	sh "./mvnw package -Pprod -DskipTests jib:dockerBuild" 
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    stage('push image') {
	withCredentials([usernamePassword(credentialsId: 'region-harbor', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
		sh "docker login -u '$USERNAME' -p $PASSWORD harbor.teco.1-4.fi.teco.online"
		sh "docker tag app:latest harbor.teco.1-4.fi.teco.online/jdemo/app:latest"
		sh "docker tag app:latest harbor.teco.1-4.fi.teco.online/jdemo/app:${env.BUILD_NUMBER}"
		sh "docker push harbor.teco.1-4.fi.teco.online/jdemo/app:latest"
		sh "docker push harbor.teco.1-4.fi.teco.online/jdemo/app:${env.BUILD_NUMBER}"
	}
	sh "echo TAG=${env.BUILD_NUMBER} > build.properties"
	archiveArtifacts artifacts: 'build.properties'
    }
}
