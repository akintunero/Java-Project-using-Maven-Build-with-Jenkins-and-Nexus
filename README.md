This is a sample Continuous Integration and Deployment pipeline set up using Jenkins to build and deploy a demo Java Maven app to Nexus on AWS.


![Image 1](https://user-images.githubusercontent.com/13016369/230787363-31019a31-35ae-4a06-b54f-2d963c766f39.png)


Step 1: Install Docker

              apt install docker.io
  

Step 2: Install Jenkins in the Docker container

            docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker  jenkins/jenkins:lts   
            
Step 3: Retrieve the default Jenkins Password

            docker exec -u 0 it <containter id> bash
            cat /var/jenkins_home/secrets/initialAdminPassword

 
Step 4: Launch Jenkins with <ip address>:8080
   

Step 5: Install the following Jenkins plugins:

            Maven Integration plugin
            Nexus Artifact Uploader plugin
            Pipeline plugin
            Git plugin
  

      

Step 6: Install Nexus
   Install Java 8 as Java 8 is compatible with Nexus 

               apt install openjdk-8-jre-headless
  
  
  
  
  
               cd /opt/
  
               wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
  
               tar -zxvf  latest-unix.tar.gz
  
               ls -l
  
               chown -R nexus:nexus nexus-3.51.0-01
  
               chown -R nexus:nexus sonatype-work
  
               vim /opt/nexus-3.51.0-01/bin/nexus.rc 
                     **** (edit the nexus.rc file and specify the username as "nexus".)

               usermod -aG sudo nexus
  
               su - nexus
  
               sudo /opt/nexus-3.51.0-01/bin/nexus start
  
               ps aux | grep nexus  (This is to check status)
  
               netstat -lnpt  (This is to check netstat)

Step 6: Launch the Nexus application using <ip-address>:8081
   
Step 7: Retrieve the default password on your Ubuntu Server using the command:
                vim /opt/sonatype-work/nexus3/admin.password
   
Step 8: Create a new Maven project in Jenkins by clicking "New Item" in the Jenkins dashboard and selecting "Maven Project". Give the project a name and click "OK".

Step 9: Configure the Maven project by setting the following properties:

      Under "Source Code Management", select "Git" and enter the repository URL for your Java Maven app hosted on GitHub or another Git hosting service.
  
      Under "Build", add the following Maven command to build your project: clean install
  
      Under "Post-build Actions", select "Deploy artifacts to Nexus" and configure the Nexus server by providing the URL, username, and password for your Nexus repository hosted on AWS.

Step 10: Create a new pipeline in Jenkins by clicking "New Item" in the Jenkins dashboard and selecting "Pipeline". Give the pipeline a name and click "OK".
   
Step 11: Configure the pipeline by entering the following script in the "Pipeline Script" box:
          
                                pipeline {
                                  agent any
                                  stages {
                                      stage('Build') {
                                          steps {
                                              sh 'mvn clean install'
                                          }
                                      }
                                      stage('Deploy') {
                                          steps {
                                              nexusArtifactUploader(
                                                  nexusVersion: 'nexus3',
                                                  protocol: 'http',
                                                  nexusUrl: 'http://<your-nexus-ip>:8081/nexus/repository/<your-repo-name>',
                                                  groupId: 'com.example',
                                                  version: '1.0-SNAPSHOT',
                                                  repository: 'snapshots',
                                                  credentialsId: 'nexus-credentials',
                                                  artifacts: [
                                                      [artifactId: 'your-artifact-id', type: 'jar', file: 'target/your-artifact-id.jar']
                                                  ]
                                              )
                                          }
                                      }
                                  }
}

  

  
 
 ***Note: This script defines a Jenkins pipeline with two stages: "Build" and "Deploy". The "Build" stage runs the Maven clean install command to build your Java Maven app. The "Deploy" stage uploads the built JAR file to your Nexus repository hosted on AWS
  
  
Step 12: Save your pipeline configuration and run your pipeline by clicking "Build Now". Jenkins will run the pipeline and upload the JAR file to your Nexus repository hosted on AWS.
  
  

