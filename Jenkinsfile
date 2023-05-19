pipeline {
  environment {
    dockerimagename = "emanico/react-app"
    dockerImage = ""
  }
  
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          serviceAccountName: sa-jenkins-cluster
          containers:
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          - name: kubectl
            image: atlassian/pipelines-kubectl:1.22.3
            command: ["sleep", "infinity"]
            tty: true
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }
  
  stages {
    
    stage('Checkout Source') {
      steps {
        container('docker') {
          git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/emanico1/jenkins-kubernetes-deployment.git'
        }
      }
    }

    stage('Build image') {
      steps{
        container('docker') {
          script {
            sh 'docker build -t $dockerimagename .'
          }
        }
      }
    }
    
    stage('Login-Docker') {
      steps {
        container('docker') {
          withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'DOCKER_REGISTRY_PWD', usernameVariable: 'DOCKER_REGISTRY_USER')]) {
            sh 'echo $DOCKER_REGISTRY_USER $DOCKER_REGISTRY_PWD'
            sh 'docker login -u $DOCKER_REGISTRY_USER -p $DOCKER_REGISTRY_PWD'
          }
        }
      }
    }
    
    stage('Push-Images-Docker-to-DockerHub') {
      steps {
        container('docker') {
          sh 'docker push $dockerimagename:latest'
        }
      }
     }
   

    stage('Deploying React.js container to Kubernetes') {
      steps {
        container('kubectl') {
	  script {
	    sh "kubectl apply -f deployment.yaml"
            sh "kubectl apply -f service.yaml"
	  }
	}
      }
    }
  }

}
