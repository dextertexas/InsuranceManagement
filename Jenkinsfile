node{
    
    def docker, tag, dockerHubUser, containerName, httpPort = ""
    
    stage('Prepare Environment'){
        echo 'Initialize Environment'
        tag="3.0"
	withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
		dockerHubUser="$dockerUser"
        } 
	containerName="insure-me"
	httpPort="8081"
    }
    
    stage('Code Checkout'){
         checkout scm
    }
    
    stage('Maven Build'){
        sh "mvn install"        
    }
	
    stage('Docker Image Build'){
        echo 'Creating Docker image'
        sh "docker build -t $dockerHubUser/$containerName:$tag --pull --no-cache ."
    }
	
    stage('SonarQube Scan'){
	withCredentials([string(credentialsId: 'SonarToken', variable: 'SonarToken')]) {
	     sh "mvn sonar:sonar -Dsonar.login=${SonarToken}"
	}
    }
	
    stage('Docker Image Scan'){
        echo 'Scanning Docker image for vulnerbilities'
        sh "trivy image ${dockerHubUser}/$containerName:${tag}"
    }   
	
    stage('Publishing Image to DockerHub'){
        echo 'Pushing the docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
			sh "docker login -u $dockerUser -p $dockerPassword"
			sh "docker push $dockerUser/$containerName:$tag"
			echo "Image push complete"
        } 
    }    
	
	stage('Docker Container Deployment'){
		sh "docker rm $containerName -f"
		sh "docker pull $dockerHubUser/$containerName:$tag"
		sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
		echo "Application started on port: ${httpPort} (http)"
	}
}



