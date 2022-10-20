pipeline {
         agent { label "linux" }
         environment {AWS_DEFAULT_REGION="us-east-1"}

         tools {  maven '3.8.6'}
        
    stages {
        stage('aws connect') {
           steps {
           withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'jenkins-aws-creds', 
           secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh '''
            aws --version
            aws ec2 describe-instances
          '''
        }
      }
}
stage("Git Clone"){
       steps{
       git branch: 'main', credentialsId: 'CODE_COMMIT_CREDENTIALS', 
       url: 'https://git-codecommit.us-east-1.amazonaws.com/v1/repos/DashboardServiceNow'
      }


}

stage('Maven Test') { 
      steps{
      sh 'mvn --version'
      dir("/var/lib/jenkins/workspace/DashboardServiceNow3006/service-now/service-now-exporter/") {
          sh 'mvn  clean test'
      }
      
     }
     }

stage('Maven Build') { 
      steps{
      sh 'mvn --version'
      dir("/var/lib/jenkins/workspace/DashboardServiceNow3006/service-now/service-now-exporter/") {
          sh 'mvn  clean package'
      }
      
     }
     }
stage('Upload to s3') {
              steps {
                  withAWS(region:'us-east-1',credentials:'jenkins-aws-creds') {
                  sh 'echo "Uploading content with AWS creds"'
                  dir("/var/lib/jenkins/workspace/DashboardServiceNow3006/service-now/service-now-exporter/service-now-exporter-api/target/") {
                   s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'service-now-exporter-api-0.0.1-SNAPSHOT.jar', bucket:'dashboardjirajarbucket/Jar/')
                  }
                  
                      
                  }
              }
    }         
     
stage('Scan') {
      steps {
        withSonarQubeEnv(installationName: 'SonarQube') { 
        dir("/var/lib/jenkins/workspace/DashboardServiceNow3006/service-now/service-now-exporter/") {
          sh 'mvn  clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.6.0.1398:sonar -Dsonar.projectKey=service-now-1 -Dsonar.java.binaries=target/sonar'
         }    
          
        }
      }
    }
    stage("Quality Gate") {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

}
}
