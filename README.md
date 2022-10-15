# DevOpsProject
DevOps Course Project
DevOps Project Deployment

1.Choose High Medium Instance like t2.large.
2.Install all necessary things on that machine like.
	$sudo yum install -y vim unzip wget git Java-11* maven 
3.Download and Install SonarQube (Running on port 9000)
	$wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.5.zip
	$unzip ~/sonarqube-7.5.zip
 -->Start the Sonar
	$ ~/sonarqube-7.5/bin/linux-x86-64/sonar.sh status
	$ ~/sonarqube-7.5/bin/linux-x86-64/sonar.sh start
	$ ~/sonarqube-7.5/bin/linux-x86-64/sonar.sh status
--> Open Sonarqube generate Token select language and copy the command for further use
	like $mvn sonar:sonar \-Dsonar.host.url=http://3.111.245.205:9000 \-Dsonar.login=b7d69f5f2c5351a6f9f84c3c527349510f34d190
4.Download and install Sonar Nexus Repository(Running on port 8081)
	$ wget https://download.sonatype.com/nexus/oss/nexus-2.14.18-01-bundle.tar.gz
	$ tar xvf nexus-2.14.18-01-bundle.tar.gz
	$ ~/nexus-2.14.18-01/bin/nexus start
5.Download and install tomcat Server.
	$ wget https://archive.apache.org/dist/tomcat/tomcat-7/v7.0.94/bin/apache-tomcat-7.0.94.tar.gz
	$ tar xvf apache-tomcat-7.0.94.tar.gz
 --> After Unzip you have to run tomcat server on different port bcoz Jenkins is running on same port(8080) that why
 --> You have to change 2 files.
	i.server.xml	ii.tomcat-users.xml
 $vim apache-tomcat-7.0.94/conf/server.xml
 --> then search /8080 after that in connector port change 8080 to 9090
	<Connector port="9090" protocols=---> save and exit file
 $vim apache-tomcat-7.0.94/conf/tomcat-users.xml
	In that you have to add user configuration add the following line in below :- 
	<tomcat-users>
	<user username="tomcat" password="tomcat" roles="manager-gui"/>
	</tomcat-users>
 --> Then start the Apache-tomcat server
	$ ~/apache-tomcat-7.0.94/bin/startup.sh	$ ~/apache-tomcat-7.0.94/bin/status.sh	$~/apache-tomcat-7.0.94/bin/shutdown.sh

5.Setting Jenkins(make sure Java-11 version is installed)Running on port 8080
	$sudo yum install -y epl-release
	$sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
	$sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
	$sudo yum -y install jenkins
	$sudo systemctl start jenkins
	$sudo cat /var/lib/jenkins/secrets/initialAdminPassword -->view password and open Jenkins
	--> select plugins to install and select none plugin for timing.

6.In Jenkins view have to create multiple jobs and create job
-->first you have to install 2 plugins I. GitHub ii. build pipeline plugging 
i).validate code(jobname):-
-->in that select source code give GitHub project link. 
	--> git link - https://github.com/Nil30/DevOpsProject.git
-->In Execute shell $mvn compile save and build job.

ii).Testing job-select for first project
-->change only shell $mvn test save and build project

iii. Sonar Analyzing
-->copy the sonar command
	$mvn sonar:sonar \-Dsonar.host.url=http://3.111.245.205:9000 \-Dsonar.login=b7d69f5f2c5351a6f9f84c3c527349510f34d190
-->save and build project

iv).Clone project in your system $git clone https://github.com/Nil30/DevOpsProject.git then move to project directory then open pom.xml file add below code like.
-->in distribution management add Ip address like
	<url>copy of nexus ip</url>
-->copy of nexus is available in nexus repo.
-->save file add file then push file to GitHub
	$git add pom.xml 
	$git commit -m "change pom.xml"
	$git push
-->Before running nexus project set the configuration 
	$sudo vim /usr/share/maven/conf/settings.xml	-->Add code in server
	<servers>
	<server>
		<id>releases</id>
		<username>admin(username of nexus)</username>
		<password>admin123(password of nexus)</password>
	</server>
-->change id and password
--> then set the job -- Artifact(jobname) and then build the project.
-->open nexus server and copy link for further use it.

vi)first move your tomcat file to opt directory
	$sudo mv apache-tomcat-7.0.94 /opt
-->then start again tomcat server $sudo ~/opt/apache-tomcat-7.0.94/bin/startup.sh
-->adding centos(user) in  jenkins 
	$sudo visudo -->search for %wheel then add user
	Jenkins ALL=(ALL) NOPASSWD: ALLL -->save the file and restart again the Jenkins
-->Deploy App(jobname) make sure you have to select git none in shell type below command
	$cat /opt/apache-tomcat-7.0.94/webapps/
	$sudo wget --user username(admin) --password password(admin123) linkofnexus(http://3.111.245.205:8081/nexus/service/local/repositories/releases/content/com/web/cal/WebAppCal/1.3.8/WebAppCal-1.3.8.war)
-->save the project and build.


7.Creating pipeline in Jenkins are two ways.
i)App pipeline
-->In Jenkins select + operation build pipeline view
--> goto Testing job->build trigger->select build after project are build->project name and like as sonar  analyses project
A.	Code validate
B.	testing
C.	Sonar Analysis
D.	Publish Artifact
E.	Deploy App
-->add also webhook trigger for first job for automation

ii. Using Pipeline
-->create pipeline script and update to github repo
-->create job using pipeline selector
-->In pipeline definition give GitHub link and also Jenkins file path save and apply. then build pipeline.
-->You have to change/conf file in localhost and then update to GitHub repo.
Jenkins File
pipeline {
    agent any

    stages {
        stage('validate') {
            steps {
                echo 'Code Validate'
                sh 'mvn compile'
            }
        }
        stage('UnitTest') {
            steps {
                echo 'Unit Testing'
                sh 'mvn test'
            }
        }
        stage('Sonar') {
            steps {
                echo 'Sonar Analysing'
                sh 'mvn sonar:sonar -Dsonar.host.url=http://3.111.245.205:9000 -Dsonar.login=b7d69f5f2c5351a6f9f84c3c527349510f34d190'
            }
        }
        stage('Public Artifact') {
            steps {
                echo 'Providing Artifact'
                sh 'mvn deploy'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying App on Tomcat Server'
                sh 'cd /opt/apache-tomcat-7.0.94/webapps/'
                sh 'sudo wget --user admin --password admin123 http://3.111.245.205:8081/nexus/service/local/repositories/releases/content/com/web/cal/WebAppCal/1.3.8/WebAppCal-1.3.8.war'
            }
        }
    }
}



