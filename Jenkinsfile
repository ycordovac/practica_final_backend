def versionPom = ""
pipeline{
	agent {
      kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: yandihlg/jenkins-nodo-java-bootcamp:latest
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
        '''
        defaultContainer 'shell'
      }
    }

    environment {
        registryCredential='yandihlg'
        registryBackend = 'yandihlg/backend-demo'
        sonarcredential='admin'

        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.58.3:8081"
        NEXUS_REPOSITORY = "backdevelop"
        NEXUS_CREDENTIAL_ID = "adminnexus"
    }
	stages {

        stage('compile app') {
          steps {
                sh "mvn clean install -DskipTests"
          }
        }

        stage('Push Image to Docker Hub') {
          steps {
            script {
              dockerImage = docker.build registryBackend + ":$BUILD_NUMBER"
              docker.withRegistry( '', registryCredential) {
                dockerImage.push()
              }
            }
          }
        }

        stage('Push Image latest to Docker Hub') {
          steps {
            script {
              dockerImage = docker.build registryBackend + ":latest"
              docker.withRegistry( '', registryCredential) {
                dockerImage.push()
              }
            }
          }
        }

        stage("Deploy to K8s"){
          steps{
                    script {
                      if(fileExists("configuracion")){
                        sh 'rm -r configuracion'
                      }
                    }

            sh 'git clone https://github.com/ycordovac/kubernetes-helm-docker-config.git configuracion --branch test-implementation'
            sh 'kubectl apply -f configuracion/kubernetes-deployment/spring-boot-app/manifest.yml -n default --kubeconfig=configuracion/kubernetes-config/config'
          }
        }

        stage("Print Java Version"){
          steps{
            script {
              javaVersion= sh 'java --version'
              echo javaVersion
              mvnVersion= sh 'mvn -version'
              echo mvnVersion
            }
          }
        }

    /**

    stage("Maven version to main branch"){
			steps{
        script {
          echo "begin when" 
          when{
            branch 'main'
            echo "1"
              echo "eliminando etiqueta snapShot de la version para la rama main"
              def pomModel = readMavenPom
              echo "2"
              def pomVersion = pomModel.getVersion().replace("-SNAPSHOT", "")
              echo "3"
              def comand='mvn versions:set -DnewVersion='+$pomVersion
              echo "4"
              sh comand
              echo "5"
              sh 'git checkout main'
              echo "6"
              sh 'git add .'
              echo "7"
              sh '''git commit -m "delete tag snapshot from maven version"'''
              echo "8"
              sh 'git push'
              echo "9"
          }

        }
			}
		}
**/
   

      stage ("Run Test") {
        steps{
          script {
              sh 'mvn test'
              junit 'target/surefire-reports/*.xml' 
          }   
        }
      }

      stage ("Cobertura Jacoco") {
        steps{
          script {
              jacoco()
          }   
        }
      }


/*
        stage('SonarQube analysis') {
          steps {
            withSonarQubeEnv(credentialsId: sonarcredential, installationName: "sonarqube-server"){
                sh "mvn clean verify sonar:sonar -DskipTests"
            }
          }
        }

        stage('Quality Gate') {
          steps {
            timeout(time: 10, unit: "MINUTES") {
              script {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
              }
            }
          }
        }
  */

      stage("Quality Tests") {
        steps {
            script {
              withSonarQubeEnv("sonarqube-server"){
                sh 'mvn clean verify sonar:sonar \
                -Dsonar.projectKey=practica-final-backend \
                -Dsonar.host.url=https://gentle-grapes-switch-213-0-57-163.loca.lt \
                -Dsonar.login=squ_d4810d6d41f2c2c6bb7a6833a7ae0971867101f3'
              }
            }
        }
		  }

      stage("Maven Build") {
            steps {
                script {
                    sh "mvn package -DskipTests=true"
                }
            }
      }

      /*
      stage("Publish to Nexus") {
        steps {
          script {
          // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
          pom = readMavenPom file: "pom.xml"
          // Find built artifact under target folder
          filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
          // Print some info from the artifact found
          echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
          // Extract the path from the File found
          artifactPath = filesByGlob[0].path
          // Assign to a boolean response verifying If the artifact name exists
          artifactExists = fileExists artifactPath
            if(artifactExists) {
              echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
              version = "${pom.version}"

              nexusArtifactUploader(
              nexusVersion: NEXUS_VERSION,
              protocol: NEXUS_PROTOCOL,
              nexusUrl: NEXUS_URL,
              groupId: pom.groupId,
              version: pom.version,
              repository: NEXUS_REPOSITORY,
              credentialsId: NEXUS_CREDENTIAL_ID,
              artifacts: [
              // Artifact generated such as .jar, .ear and .war files.
              [artifactId: pom.artifactId,
              classifier: "",
              file: artifactPath,
              type: pom.packaging],
              // Lets upload the pom.xml file for additional information for Transitive dependencies
              [artifactId: pom.artifactId,
              classifier: "",
              file: "pom.xml",
              type: "pom"]
              ])
            } else {
              error "*** File: ${artifactPath}, could not be found"
            }
          }
        }
		  }
      */
    }
    
      

	post {
		always {
			sh "docker logout"
		}
	}

}