pipeline {      
	agent any            
	tools {          
		jdk 'jdk17'          
		maven 'maven3'              
		}            
		environment {          
			SCANNER_HOME = tool 'sonar-scanner'
			JAVA_HOME = tool 'jdk17'
		}        
		stages {          
			stage('Git Checkout') {              
				steps {                  
					git branch: 'main', url: 'https://github.com/NachiketDamle01/cicd-project.git'             
				}          
			}                    
			stage('Compile') {              
				steps {                  
					sh "mvn compile"              
				}          
			}                    
			stage('Test') {              
				steps {                  
					sh "mvn package -DskipTests=true"              
				}          
			}                    
			stage('Trivy Scan File System') {              
				steps {                  
					sh "trivy fs --format table -o trivy-fs-report.html ."              
				}          
			}                    
			// stage('SonarQube Analysis') {              
			// 	steps {                  
			// 		withSonarQubeEnv('sonar') {                      
			// 			sh '''${SCANNER_HOME}/bin/sonar-scanner                       
			// 			-Dsonar.projectKey=cicd-project                          
			// 			-Dsonar.projectName=cicd-project                          
			// 			-Dsonar.java.binaries=target/classes
			// 			-Dsonar.sources=src/main/java'''                 
			// 		}              
			// 	}          
			// }                    
			stage('Build') {              
				steps {                  
					sh "mvn package -DskipTests=true"              
				}          
			}          
			stage('Deploy Artifacts To Nexus') {              
				steps {                  
					withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {                      
						sh "mvn deploy -DskipTests=true"                  
					}              
				}          
			}
			stage('Verify Docker') {
            			steps {
                			sh 'docker --version'
                			sh 'docker info'
                			sh 'docker run hello-world'
            			}
       	 		}
			stage('Build & Tag Docker Image') {              
				steps {                  
					script {                      
						withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {                          
							sh "docker build -t 2121997/cicd-project:latest ."                      
						}                  
					}              
				}          
			}                    
			stage('Trivy Scan Image') {              
				steps {                  
					sh "trivy image --format table -o trivy-image-report.html 2121997/cicd-project:latest"              
				}          
			}                    
			stage('Publish Docker Image') {              
				steps {                  
					script {                      
						withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {                          
							sh "docker push 2121997/cicd-project:latest"                      
						}                  
					}              
				}          
			}         
			stage('Deploy to EKS') {              
				steps {                  
					withKubeConfig(caCertificate: '', clusterName: 'my-eks7', contextName: '', credentialsId: 'k8_token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://4C8969F8547FF883EC1218A5A4FEF091.gr7.ap-south-1.eks.amazonaws.com') {
						sh "kubectl apply -f ds.yml -n webapps"
						sleep 60                  
					}             
				}          
			}          
			stage('Verify deployment') {              
				steps {                  
					withKubeConfig(caCertificate: '', clusterName: 'my-eks7', contextName: '', credentialsId: 'k8_token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://4C8969F8547FF883EC1218A5A4FEF091.gr7.ap-south-1.eks.amazonaws.com') {
						sh "kubectl get pods -n webapps"                      
						sh "kubectl get svc -n webapps"                  
						}             
					}          
				}      
			}      
			post {          
				always {              
					script {                  
						def jobName = env.JOB_NAME                  
						def buildNumber = env.BUILD_NUMBER                  
						def pipelineStatus = currentBuild.result ?: 'UNKNOWN'                  
						def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'                    
						def body = """                      
						<html>                      
						<body>                      
						<div style="border: 4px solid ${bannerColor}; padding: 10px;">                      
						<h2>${jobName} - Build ${buildNumber}</h2>                      
						<div style="background-color: ${bannerColor}; padding: 10px;">                      
						<h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>                      
						</div>                      
						<p>Check the <a href="${BUILD_URL}">console output</a>.</p>                      
						</div>                      
						</body>                      
						</html>                  
						"""                    
					emailext (                      
						subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",                      
						body: body,                      
						to: 'nachiketdamle01@gmail.com',                      
						from: 'jenkins@example.com',                      
						replyTo: 'jenkins@example.com',                      
						mimeType: 'text/html',                      
						attachmentsPattern: 'trivy-image-report.html'                  
					)              
				}          
			}      
		}  
	}  
