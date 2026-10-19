pipeline {
    agent any
    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }
    triggers { pollSCM('* * * * *') }
    environment {
        BACK_IMAGE  = 'nourchokri/gestion-projets-backend'
        FRONT_IMAGE = 'nourchokri/gestion-projets-frontend'
    }
    stages {
        stage('Récupération du code') {
            steps { checkout scm }
        }
        stage('Création du livrable') {
            steps { dir('backend') { sh 'mvn clean package -DskipTests' } }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t ${BACK_IMAGE}:${BUILD_NUMBER} -t ${BACK_IMAGE}:latest ./backend"
                sh "docker build -t ${FRONT_IMAGE}:${BUILD_NUMBER} -t ${FRONT_IMAGE}:latest ./frontend"
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
                    sh "docker push ${BACK_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${BACK_IMAGE}:latest"
                    sh "docker push ${FRONT_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${FRONT_IMAGE}:latest"
                }
            }
        }
    }
    post {
        success { archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true }
        failure {
            mail to: 'nour.chokri@esprit.tn',
                 subject: "ÉCHEC du build : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Le build a échoué.\n\nConsulter les logs : ${env.BUILD_URL}console"
        }
        always { sh 'docker logout || true' }
    }
}
