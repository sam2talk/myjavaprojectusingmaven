pipeline {
  agent any
  
  tools {
  maven 'Maven3'
  }
  stages {
    stage ('Build') {
      steps {
      sh 'mvn clean install -f MyWebApp/pom.xml'
      }
    }
    stage ('Code Quality') {
      steps {
        withSonarQubeEnv('SonarQube') {
        sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
        }
      }
    }
    stage ('JaCoCo') {
      steps {
      jacoco()
      }
    }
    stage ('Nexus Upload') {
      steps {
      nexusArtifactUploader(
      nexusVersion: 'nexus3',
      protocol: 'http',
      nexusUrl: 'ec2-44-204-174-106.compute-1.amazonaws.com:8081',
      groupId: 'myGroupId',
      version: '1.0-SNAPSHOT',
      repository: 'maven-snapshots',
      credentialsId: 'b27330be-9c76-414e-a263-235610b45254',
      artifacts: [
      [artifactId: 'MyWebApp',
      classifier: '',
      file: 'MyWebApp/target/MyWebApp.war',
      type: 'war']
      ])
      }
    }
    stage ('DEV Deploy') {
      steps {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: '43b25625-bec4-4eeb-b4b8-e5079aa76e9d', path: '', url: 'http://ec2-44-203-10-76.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
      }
    }
    stage ('Slack Notification') {
      steps {
        echo "deployed to DEV Env successfully"
        slackSend(channel:'cicd-jan23', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }


    stage ('DEV Approve') {
      steps {
      echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }
     stage ('QA Deploy') {
      steps {
        echo "deploying to QA Env "
        deploy adapters: [tomcat9(credentialsId: '43b25625-bec4-4eeb-b4b8-e5079aa76e9d', path: '', url: 'http://ec2-44-203-10-76.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
        }
    }
    stage ('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
        }
      }
    }


    stage ('Slack Notification for QA Deploy') {
      steps {
        echo "deployed to QA Env successfully"
        slackSend(channel:'cicd-jan23', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }  
  }
}
