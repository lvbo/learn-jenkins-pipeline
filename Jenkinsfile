pipeline {
    agent any

    tools {
        maven 'mvn-3.5.4'
    }
    
    stages {
        stage('Build') {
            steps {
                configFileProvider([configFile(fileId: 'maven-global-setting', variable: 'MAVEN_GLOBAL_ENV')]) {
                    sh "mvn -s $MAVEN_GLOBAL_ENV clean package spring-boot:repackage"
                    // sh "printenv"
                }
            }
        }
    }
}
