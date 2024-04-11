COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    tools {
        maven "Maven-3.9.6"
        jdk "jdk17"
    }

    environment {
        SANNER_HOME = tool 'Sonar-5'
    }

    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/gitgo23/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''$SANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=.'''
                }
            }
        }

        // stage('Owasp Dependency Check') {
        //     steps {
        //         dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'Maven-3.9.6', mavenSettingsConfig: '', traceability: true) {
                   sh "mvn deploy -DskipTests=true" 
                }
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications'
            slackSend channel: 'jenkins-slack-notification', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "${currentBuild.currentResult}: Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            
            // Send email notification
            emailext subject: "Pipeline ${currentBuild.result}: ${env.JOB_NAME}",
                      body: "Build ${env.BUILD_NUMBER} ${currentBuild.result}\n\nCheck console output at: ${env.BUILD_URL}",
                      to: "enoch3054@gmail.com",
                      attachLog: true
        }
    }
}

