pipeline {
 environment { 
        registry = "testvikrams1/numeric-app" 
        registryCredential = 'dockerhub_id' 
        dockerImage = '' 
        deploymentName = "devsecops"
        containerName = "devsecops-container"
        serviceName = "devsecops-svc"
        imageName = "testvikrams1/numeric-app:${GIT_COMMIT}"
        applicationURL = "http://devsecops-demov.eastus.cloudapp.azure.com/"
        applicationURI = "/increment/99"
    }
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
    }
    
    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
    }
    
     //    stage('Vulnerability Scan - Docker ') {
    //      steps {
    //         sh "mvn dependency-check:check"   
    //        }
    // }

    stage('Vulnerability Scan - Docker') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            sh "bash trivy-docker-image-scan.sh"
          }
        )
      }
    }
   
    
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: registryCredential, url: ""]) {
          sh 'printenv'
          sh 'docker build -t testvikrams1/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push testvikrams1/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    stage('Vulnerability Scan - Kubernetes') {
      steps {
        parallel(
          "OPA Scan": {
            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
          },
          "Kubesec Scan": {
            sh "bash kubesec-scan.sh"
          }
        )
      }
    }
    
    //stage('Kubernetes Deployment - DEV') {
     // steps {
      //  withKubeConfig([credentialsId: 'kubeconfig']) {
      //    sh "sed -i 's#replace#testvikrams1/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
       //   sh "kubectl apply -f k8s_deployment_service.yaml"
       // }
      //}
  //  }
 // }
 
      stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment.sh"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }
    
 stage('Integration Tests - DEV') {
      steps {
        script {
          try {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash integration-test.sh"
            }
          } catch (e) {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "kubectl -n default rollout undo deploy ${deploymentName}"
            }
            throw e
          }
        }
      }
    }

 stage('OWASP ZAP - DAST') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'bash zap.sh'
        }
      }
  }

  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
     // publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP', reportTitles: 'OWASP'])
    }

    // success {

    // }

    // failure {

    // }
  }

}