pipeline {
  agent any 
  tools {
    maven 'Maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
      sh 'rm trufflehog || true'
      sh 'docker run gesellix/trufflehog --json https://github.com/etho0/webapp.git > trufflehog'
      sh 'cat trufflehog'
      }
    }
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/etho0/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/workspace/webapp-cicd-pipeline@2/odc-reports/dependency-check-report.xml'
        
      }
    }
    
    
    stage('SAST-Semgrep-Scan') {
        steps {
          sh 'pip3 install semgrep'
          sh 'semgrep --config p/ci --config p/security-audit --config p/secrets --output scan_results.json --json'
          sh 'cat scan_results.json'
      }
    }

            
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@65.2.172.6:/home/ubuntu/prod/apache-tomcat-10.1.6/webapps/WebApp.war'
              }      
           }       
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.2.142.213 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://65.2.172.6:8080/WebApp/" || true'
        }
      }
    }
    
  }
}
