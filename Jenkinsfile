pipeline {
    agent any
    tools{
        jdk 'jdk'
        maven 'maven'
    }
    environment {
        aws_region = "us-west-1"   //ur aws region
        eks_cluster = "myeks"  // ur cluster
    }
	
    stages {
        stage('checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Gowtham-745/vulnado.git'  //update your repo
            }
        }
		stage('OWASP Dependency-Check') {
			steps {
				dependencyCheck additionalArguments: ''' 
				-o './'
				-s './'
				-f 'ALL' 
				--prettyPrint''', odcInstallation: 'owasp'
				dependencyCheckPublisher pattern: 'dependency-check-report.xml'
			}
		}
        stage('Clean') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/Gowtham-745/vulnado.git'

                // To run Maven on a Windows agent, use
                sh "mvn clean"
            }
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Cleaning Project is Done'
                }
            }
        }
		
        stage('Compile') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/Gowtham-745/vulnado.git'
                // To run Maven on a Windows agent, use
                sh "mvn compile"
            }
			
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                   echo 'Compiling Project is Done'
                }
            }
        }

        stage('Snyk Security') {
            steps {
                snykSecurity failOnError: false, failOnIssues: false, organisation: 'attiligowtham', projectName: 'vulnado', snykInstallation: 'snyk', snykTokenId: 'snyk-id', targetFile: 'pom.xml'
                //bat 'C:\\Users\\attil\\AppData\\Roaming\\npm\\node_modules\\snyk\\wrapper_dist\\snyk-win.exe test'
            }
        }

		stage('Package') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/Gowtham-745/vulnado.git'
                // To run Maven on a Windows agent, use
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
			
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                 archiveArtifacts 'target/*.jar'
                }
            }
        }
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                        sh "docker tag vulnado_vulnado gowtham47/vuln:latest "
                        sh "docker push gowtham47/vuln:latest "
                        sh "docker tag vulnado_client gowtham47/clin:latest "
                        sh "docker push gowtham47/clin:latest "
                        sh "docker tag vulnado_internal_site gowtham47/ins:latest "
                        sh "docker push gowtham47/ins:latest "
                        sh "docker tag postgres gowtham47/d-b:latest "
                        sh "docker push gowtham47/d-b:latest "
                    }
                }
            }
        }
        
        stage('deploy') {
            steps {
                script {
                    sh "aws eks --region $aws_region update-kubeconfig --name $eks_cluster"
                    sh "kubectl get svc"
                }
            }
        }
    }
}
