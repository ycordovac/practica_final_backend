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
            if(fileExists("practica_final_backend")){
                sh 'rm -r practica_final_backend'
            }
            sh 'mvn clean compile test'
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