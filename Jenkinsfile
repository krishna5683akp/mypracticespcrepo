pipeline {
    agent { label 'OPENJDK' }
    triggers { pollSCM('* * * * *') }
    stages {
        stage('git') {
            steps {
                git branch: 'dev', url: 'https://github.com/krishna5683akp/mypracticespcrepo.git'              
            }
        }
        stage ('Artifactory configuration') {
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "JFROG_ID",
                    releaseRepo: "fortetsingrepo-libs-release-local",
                    snapshotRepo: "fortetsingrepo-libs-snapshot-local"
                )
            }
        }
        stage ('Exec Maven') {
            steps {
                rtMavenRun (
                    tool: 'MAVEN_TOOL', // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER"
                )
            }
        }
        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "JFROG_ID"
                )
            }
        }
        stage ('SoanrQube Analysis') {
            steps { 
                withSonarQubeEnv('SONARQUBE') {
                    sh 'mvn clean package sonar:sonar'
                }
            }
        }
        stage('docker image') {
            steps {
                sh """docker image build -t spc:1.0 .
                    docker image tag spc:1.0 fortestingmyself.jfrog.io/myrepo/spc:1.0
                    docker push fortestingmyself.jfrog.io/myrepo/spc:1.0"""
            }
        }
    }
    post {
        success {
            junit '**/surefire-reports/*.xml'
        }
    }
}
