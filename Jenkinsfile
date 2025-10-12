pipeline {
    agent any
    environment {
        MAVEN_OPTS = "--add-opens java.base/java.lang=ALL-UNNAMED"
    }

    stages {
        stage('Build with maven') {
            steps {
                sh 'cd SampleWebApp && mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'cd SampleWebApp && mvn test'
            }
        }

        stage('Code Quality Scan') {
            steps {
                withSonarQubeEnv('sonarscan') {
                    sh "mvn -f SampleWebApp/pom.xml sonar:sonar"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('push to nexus') {
            steps {
                nexusArtifactUploader artifacts: [[
                    artifactId: 'SampleWebApp',
                    classifier: '',
                    file: 'SampleWebApp/target/SampleWebApp.war',
                    type: 'war'
                ]],
                credentialsId: 'nexus',
                groupId: 'SampleWebApp',
                nexusUrl: 'http://ec2-204-236-200-94.compute-1.amazonaws.com:8081',
                nexusVersion: 'nexus2',
                protocol: 'http',
                repository: 'maven-snapshots',
                version: '1.0-SNAPSHOT'
            }
        }

        stage('deploy to tomcat') {
            steps {
                deploy adapters: [
                    tomcat9(
                        alternativeDeploymentContext: '',
                        credentialsId: 'tompass',
                        path: '',
                        url: 'http://50.17.27.181:8080/'
                    )
                ],
                contextPath: 'monolithicApp',
                war: '**/*.war'
            }
        }
    }
}
