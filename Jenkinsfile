pipeline {
    agent any

    tools {
        maven 'mvn-3.5.4'
    }

    triggers {
        // cron('45 18 * * *')
        upstream(upstreamProjects: 'learn-jenkins-pipeline-upstream', threshold: hudson.model.Result.SUCCESS)
    }
    
    stages {
        stage('clean') {
            steps {
                sh "mvn clean"
            }
        }
        stage('pmd') {
            steps {
                sh "mvn pmd:pmd"
            }
        }
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                    mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.4.1.1168:sonar \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build') {
            steps {
                configFileProvider([configFile(fileId: 'maven-global-setting', variable: 'MAVEN_GLOBAL_ENV')]) {
                    sh "mvn -s $MAVEN_GLOBAL_ENV package spring-boot:repackage"
                    // sh "printenv"
                }
            }
        }
        stage('Jacoco') {
            steps {
               jacoco(
                   // 代码覆盖率统计文件位置，Ant风格路径表达式
                   execPattern: 'target/**/*.exec',
                   // classes文件位置，Ant风格路径表达式
                   classPattern: 'target/classes',
                   // 源码文件位置，Ant风格路径表达式
                   sourcePattern: 'src/main/java',
                   // 排除分析的位置，Ant风格路径表达式
                   exclusionPattern: 'src/test*',
                   // 是否禁用每行覆盖率的源文件显示
                   skipCopyOfSrcFiles: false,
                   changeBuildStatus: false,
                   // 字节码指令覆盖率
                   minimumInstructionCoverage: '30', maximumInstructionCoverage: '70',
                   // 行覆盖率
                   minimumLineCoverage: '30', maximumLineCoverage: '70',
                   // 圈复杂度覆盖率
                   minimumComplexityCoverage: '30', maximumComplexityCoverage: '70',
                   // 方法覆盖率
                   minimumMethodCoverage: '30', maximumMethodCoverage: '70',
                   // 类覆盖率
                   minimumClassCoverage: '30', maximumClassCoverage: '70',
                   // 分支覆盖率
                   minimumBranchCoverage: '30', maximumBranchCoverage: '70',
                   buildOverBuild: false,
                   // 以下是各个维度覆盖率的变化阈值
                   deltaInstructionCoverage: '80', deltaLineCoverage: '80',
                   deltaMethodCoverage: '80', deltaClassCoverage: '80',
                   deltaComplexityCoverage: '80', deltaBranchCoverage: '80'
               )
            }
        }
        stage("Reports") {
            steps {
                allure results: [[path: 'target/allure-results']]
            }
        }
    }

    post {
        always {
            pmd(canRunOnFailed:true, pattern: '**/target/pmd.xml')
            // junit testResults: "**/target/surefire-reports/*.xml"
            // cleanWs()
        }
    }
}
