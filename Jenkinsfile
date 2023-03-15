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
      sh 'docker run gesellix/trufflehog --json https://github.com/etho0/webapp.git > trufflehog.json'
      sh 'cat trufflehog.json'
      sh 'curl -X "POST" "http://13.234.59.184:8080/api/v2/reimport-scan/" -H "Authorization: Token 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -H "accept: application/json" -H "Content-Type: multipart/form-data" -H "X-CSRFToken: 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -F "minimum_severity=Info" -F "active=true" -F "verified=true" -F "do_not_reactivate=true" -F "scan_type=Trufflehog Scan" -F "file=@trufflehog.json;type=application/json" -F "product_name=WebApp-Pipeline" -F "engagement_name=Devsecops" -F "test=1" -F "test_title=Trufflehog (Trufflehog Scan)" -F "push_to_jira=false" -F "close_old_findings=true" -F "close_old_findings_product_scope=false" -F "create_finding_groups_for_all_findings=true"'
      }
    }
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/etho0/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/workspace/webapp-cicd-pipeline@2/odc-reports/dependency-check-report.xml'
         sh 'curl -X "POST" "http://13.234.59.184:8080/api/v2/reimport-scan/" -H "Authorization: Token 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -H "accept: application/json" -H "Content-Type: multipart/form-data" -H "X-CSRFToken: 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -F "minimum_severity=Info" -F "active=true" -F "verified=true" -F "do_not_reactivate=true" -F "scan_type=Dependency Check Scan" -F "file=@/var/lib/jenkins/workspace/webapp-cicd-pipeline@2/odc-reports/dependency-check-report.xml;type=application/json" -F "product_name=WebApp-Pipeline" -F "engagement_name=Devsecops" -F "test=3" -F "test_title=OWASP Dependency checks (Dependency Check Scan)" -F "push_to_jira=false" -F "close_old_findings=true" -F "close_old_findings_product_scope=false" -F "create_finding_groups_for_all_findings=true"'
        
      }
    }
    
    
    stage('SAST-Semgrep-Scan') {
        steps {
          sh 'pip3 install semgrep'
          sh 'semgrep --config p/ci --config p/security-audit --config p/secrets --output scan_results.json --json'
          sh 'cat scan_results.json'
          sh 'curl -X "POST" "http://13.234.59.184:8080/api/v2/reimport-scan/" -H "Authorization: Token 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -H "accept: application/json" -H "Content-Type: multipart/form-data" -H "X-CSRFToken: 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -F "minimum_severity=Info" -F "active=true" -F "verified=true" -F "do_not_reactivate=true" -F "scan_type=Semgrep JSON Report" -F "file=@scan_results.json;type=application/json" -F "product_name=WebApp-Pipeline" -F "engagement_name=Devsecops" -F "test=2" -F "test_title=semgrep (Semgrep JSON Report)" -F "push_to_jira=false" -F "close_old_findings=true" -F "close_old_findings_product_scope=false" -F "create_finding_groups_for_all_findings=true"'
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
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.2.142.213 "docker run -v $(pwd):/zap/wrk/:rw --user root -t owasp/zap2docker-stable zap-baseline.py -t http://65.2.172.6:8080/WebApp/ -x zap-report.xml" || true'
         sh 'scp ubuntu@65.2.142.213:/var/lib/jenkins/workspace/webapp-cicd-pipeline@2/zap-report.xml zap-report.xml'
         sh 'curl -X "POST" "http://13.234.59.184:8080/api/v2/reimport-scan/" -H "Authorization: Token 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -H "accept: application/json" -H "Content-Type: multipart/form-data" -H "X-CSRFToken: 5eb4cf9d009841b5b4c7c27ce86b9f78b3dac3d0" -F "minimum_severity=Info" -F "active=true" -F "verified=true" -F "do_not_reactivate=true" -F "scan_type=ZAP Scan" -F "file=@zap-report.xml;type=application/json" -F "product_name=WebApp-Pipeline" -F "engagement_name=Devsecops" -F "test=4" -F "test_title=Zap Baseline Scan (ZAP Scan)" -F "push_to_jira=false" -F "close_old_findings=true" -F "close_old_findings_product_scope=false" -F "create_finding_groups_for_all_findings=true"'
        }
      }
    }
    
  }
}
