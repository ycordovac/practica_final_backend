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

        stage('Build app') {
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

		stage("Deploy to K8s")
		{
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

    stage("Print Java Version")
		{
			steps{
                script {
                  javaVersion= sh 'java --version'
                  echo javaVersion
                  System.err.println(javaVersion)
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