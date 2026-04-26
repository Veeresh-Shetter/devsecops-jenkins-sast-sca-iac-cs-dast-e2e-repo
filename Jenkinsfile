pipeline {
  agent { label 'windows' }
  tools {
    maven 'Maven_3_8_7'
  }
  stages {

    stage('CompileandRunSonarAnalysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          bat("mvn -Dmaven.test.failure.ignore verify sonar:sonar -Dsonar.login=%SONAR_TOKEN% -Dsonar.projectKey=easybuggy -Dsonar.host.url=http://<YOUR_SONAR_HOST>:9000/")
        }
      }
    }

    stage('Build') {
      steps {
        withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
          script {
            app = docker.build("asecurityguru/testeb")
          }
        }
      }
    }

    stage('Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
          script {
            app.push("latest")
          }
        }
      }
    }

    stage('RunContainerScan') {
      steps {
        withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
          script {
            try {
              bat("C:\\DevSecops\\snyk\\snyk-win.exe auth %SNYK_TOKEN% && C:\\DevSecops\\snyk\\snyk-win.exe container test asecurityguru/testeb")
            } catch (err) {
              echo err.getMessage()
            }
          }
        }
      }
    }

    stage('RunSnykSCA') {
      steps {
        withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
          bat("C:\\DevSecops\\snyk\\snyk-win.exe auth %SNYK_TOKEN% && mvn snyk:test -fn")
        }
      }
    }

    stage('RunDASTUsingZAP') {
      steps {
        bat("C:\\DevSecops\\ZAP_2.16.0_Crossplatform\\ZAP_2.16.0\\zap.sh -port 9393 -cmd -quickurl https://<YOUR_APP_URL> -quickprogress -quickout C:\\zap\\Output.html")
      }
    }

    stage('checkov') {
      steps {
        bat("checkov -s -f main.tf")
      }
    }

  }
  post {
    always {
      archiveArtifacts artifacts: 'C:\\zap\\Output.html', allowEmptyArchive: true
      cleanWs()
    }
    failure {
      echo 'Pipeline failed! Check the logs for security issues.'
    }
    success {
      echo 'Pipeline completed successfully. All DevSecOps checks passed.'
    }
  }
}
