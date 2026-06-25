pipeline {


 agent any
  tools {
    maven 'MAVEN_3_9_11'
    jdk 'JDK_26'
  }
	environment {
		REGISTRY_USER = "renzor111" 
        IMAGE_NAME = "renzor111/retail-store-u20231d974"
        TAG        = "${env.BUILD_NUMBER}" 
    }

stages {
    stage ('Compile Project') {
      steps {
      docker.image('maven:3.9.11-eclipse-temurin-26-alpine').inside {
        sh 'mvn clean compile'
        }
      }
    }


    stage('Validate Checkstyle') {
      steps {
        withMaven(maven: 'MAVEN_3_9_11') {
          sh 'mvn checkstyle:check'
        }
      }
    }

   stage('Validate Unit Tests') {
  steps {
    docker.image('maven:3.9.11-eclipse-temurin-26-alpine').inside {
        sh 'mvn test'
    }
  }
}

    stage('Validate Test Coverage') {
      steps {
        withMaven(maven: 'MAVEN_3_9_11') {
          sh 'mvn clean verify jacoco:report'
          sh 'mvn jacoco:check'
        }
      }
    }

	 stage ('SonarQube Analysis') {
        steps {
            withSonarQubeEnv('MiSonarServer') {
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=retail-store'
            }
		
	        script {
	            timeout(time: 10, unit: 'MINUTES') { 
	             
	                def qg = waitForQualityGate()
	                
	                
	                if (qg.status != 'OK') {
	                    error "El pipeline se ha detenido porque el código no superó el Quality Gate de SonarQube. Estado: ${qg.status}"
	                }
	            }
	        }

        }
     }

	 
	  stage('Construir y Publicar Imagen Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        echo "Iniciando sesión en Docker Hub..."
                        sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"

                        echo "Construyendo imagen optimizada AMD64..."
                        sh "docker build --platform linux/amd64 -t ${REGISTRY_USER}/${IMAGE_NAME}:${TAG} -t ${REGISTRY_USER}/${IMAGE_NAME}:latest ."
                    }
                }
            }
        }
	  




    }
  

}
