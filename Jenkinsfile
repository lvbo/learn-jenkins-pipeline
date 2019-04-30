pipeline {
    agent any

    tools {
        maven 'mvn-3.5.4'
    }
    
    stages {
        stage('pmd') {
            steps {
                sh "mvn pmd:pmd"
            }
        }
        stage('Build') {
            steps {
                configFileProvider([configFile(fileId: 'maven-global-setting', variable: 'MAVEN_GLOBAL_ENV')]) {
                    sh "mvn -s $MAVEN_GLOBAL_ENV clean package spring-boot:repackage"
                    // sh "printenv"
                }
            }
        }
    }

    post {
        always {
            pmd(canRunOnFailed:true, pattern: '**/target/pmd.xml')
            junit testResults: "**/target/surefire-reports/*.xml"
            cleanWs()
        }
    }
}
