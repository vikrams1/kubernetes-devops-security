pipeline {
 environment { 
        registry = "testvikrams1/numeric-app" 
        registryCredential = 'dockerhub_id' 
        dockerImage = '' 
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
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }
    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
           sh "mvn sonar:sonar Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demov.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_c8baaad1f91feec3d1ef3f5bd2c78fba346b9aeb"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
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
    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#testvikrams1/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
 }