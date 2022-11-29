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
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - cat
    imagePullPolicy: IfNotPresent
    tty: true
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

        DOCKERHUB_ID="yandihlg"
        DOCKERHUB_CREDENTIALS=credentials("yandihlg")
        DOCKER_IMAGE_NAME="yandihlg/practica-final-backend"

        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.49.4:8081"
        NEXUS_REPOSITORY = "backdevelop"
        NEXUS_CREDENTIAL_ID = "adminnexus"
        version=null
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


      stage("Quality Tests") {
        steps {
            script {
              withSonarQubeEnv("sonarqube-server"){
                sh 'mvn clean verify sonar:sonar \
                -Dsonar.projectKey=practica-final-backend \
                -Dsonar.host.url=http://192.168.49.3:9000 \
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

      stage("Publish to Nexus") {
        steps {
          script {
          pom = readMavenPom file: "pom.xml"
          filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
          echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
          artifactPath = filesByGlob[0].path
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

      stage("Build & Push"){
			steps { 
				container('shell'){
					script {
						pom = readMavenPom(file: 'pom.xml')
						version = pom.version
					}
				}
				container('kaniko'){
					script {
						withCredentials([usernamePassword(credentialsId: "yandihlg", passwordVariable: "yandihlgPassword", usernameVariable: "yandihlgUser")]) {
							AUTH = sh(script: """echo -n "${env.yandihlgUser}:${env.yandihlgPassword}" | base64""", returnStdout: true).trim()
							command = """echo '{"auths": {"https://index.docker.io/v1/": {"auth": "${AUTH}"}}}' >> /kaniko/.docker/config.json"""
							sh("""
							set +x
							${command}
							set -x
							""")
							sh "/kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination ${DOCKER_IMAGE_NAME}:${version} --cleanup"
						}
					}
				}
			}
		}
  }    

	post {
		always {
			sh "docker logout"
		}
	}

}