pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ScaleSec/vulnado.git'
            }
        }
        stage('Build') {
            steps {
                sh '''
                /var/jenkins_home/apache-maven-3.6.3/bin/mvn --batch-mode -V -U -e clean compile \
                verify -Dsurefire.useFile=false -Dmaven.test.failure.ignore
                '''
            }
        }
        stage('Analysis') {
            steps {
                sh '''
                /var/jenkins_home/apache-maven-3.6.3/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle \
                pmd:pmd pmd:cpd findbugs:findbugs
                '''
            }
        }
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'SSDSONARQUBE'
            }
            steps {
                withSonarQubeEnv('SSDSONARQUBE') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=OWASP -Dsonar.host.url=http://172.18.0.2:9000 -Dsonar.sources=. -Dsonar.java.binaries=target/classes"
                }
            }
        }
    }
    post {
        always {
            junit testResults: '**/target/surefire-reports/TEST-*.xml'
            recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
            recordIssues enabledForFailure: true, tools: [checkStyle(pattern: '**/target/checkstyle-result.xml')]
            recordIssues enabledForFailure: true, tools: [spotBugs(pattern: '**/target/findbugsXml.xml')]
            recordIssues enabledForFailure: true, tools: [cpd(pattern: '**/target/cpd.xml')]
            recordIssues enabledForFailure: true, tools: [pmdParser(pattern: '**/target/pmd.xml')]
        }
    }
}
