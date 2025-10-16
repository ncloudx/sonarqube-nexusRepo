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
                withSonarQubeEnv('sonarscanner') {
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
               nexusArtifactUploader artifacts: [[artifactId: 'SampleWebApp', classifier: '', file: 'SampleWebApp/target/SampleWebApp.war', type: 'war']], credentialsId: 'nexus', groupId: 'SampleWebApp', nexusUrl: 'ec2-3-93-162-207.compute-1.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-snapshot'
            }
        }

        stage('deploy to tomcat') {
            steps {
                deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tompass', path: '', url: 'http://3.85.206.45:8080/')], contextPath: 'myapp', war: '**/*.war'
            }
        }
    }
}
