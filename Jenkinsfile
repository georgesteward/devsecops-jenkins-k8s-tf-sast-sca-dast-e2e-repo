pipeline {
        agent any
        
        environment {
            PRISMA_API_URL="https://api.prismacloud.io"
        }
        tools { 
        maven 'Maven_3_2_5'  
        }
        stages {
           stage('CompileandRunSonarAnalysis') {
             steps {	
		sh 'mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar -Dsonar.projectKey=geo123 -Dsonar.organization=geo123 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=2541c79874304bbbb38fa3d633dce25a346e33c3'
	     }
           }

            // stage('Checkout') {
            //   steps {
            //       git branch: 'main', url: 'https://github.com/georgesteward/devsecops-jenkins-k8s-tf-sast-sca-sonarcloud-snyk-repo.git'
            //       stash includes: '**/*', name: 'source'
            //   }
            // }
            // stage('Checkov') {
            //     steps {
            //         withCredentials([string(credentialsId: 'PC_USER', variable: 'pc_user'),string(credentialsId: 'PC_PASSWORD', variable: 'pc_password')]) {
            //             script {
            //                 docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''") {
            //                   unstash 'source'
            //                   try {
            //                       sh 'checkov -d . --use-enforcement-rules -s -o cli -o junitxml --output-file-path console,results.xml --bc-api-key ${pc_user}::${pc_password} --repo-id  georgesteward/devsecops-jenkins-k8s-tf-sast-sca-sonarcloud-snyk-repo --branch main'
            //                       junit skipPublishingChecks: true, testResults: 'results.xml'
            //                   } catch (err) {
            //                       junit skipPublishingChecks: true, testResults: 'results.xml'
            //                       throw err
            //                   }
            //                 }
            //             }
            //         }
            //     }
            // }
           stage('RunSCAAnalysisUsingSnyk') {
             steps {		
		withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
		   sh 'mvn snyk:test -fn'
		   }
	     }
    	   }
        stage('SnykMonitor') {
             steps {		
		withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
		   sh 'mvn snyk:monitor -fn'
		   }
	     }
    	   }
	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("asg")
                 }
               }
            }
    	  }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://853504916271.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	  }
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}
	   
	stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -port 9000 -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
       } 
  }
}
