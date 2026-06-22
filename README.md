PROGRAM 7
STEPS FOR EXECUTION
1.	Check ansible installation: ansible --version
2.	Install Ansible if it is not installed
a.	sudo apt update
b.	wget https://bootstrap.pypa.io/get-pip.py
c.	sudo apt install python3-pip -y
d.	python3 -m pip install --user ansible  (or) python3 -m pip install --user ansible --break-system-packages (if d does not work)
3.	Set PATH (Temporary)
export PATH=$HOME/.local/bin:$PATH
4.	Set PATH (Permanent) and reload bashrc
echo ‘export PATH=$HOME/.local/bin:$PATH’ >> ~/.bashrc
source ~/.bashrc
5.	Set Locale
export LC_ALL=C.UTF-8
export LANG=C.UTF-8
6.	Verify installation: ansible –version
7.	Create Playbook: nano setup.yml
 
8.	Create Inventory File: nano hosts.ini
 
9.	Execute Playbook: ansible-playbook -I hosts.ini setup.yml

PROGRAM 8
1.	Install JDK
a.	wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
b.	tar -xvzf jdk-21_linux-x64_bin.tar.gz
c.	mv jdk-21.0.10 ~/jdk21
d.	mv jdk-21.0.10 ~/jdk21
e.	export PATH=$JAVA_HOME/bin:$PATH
f.	java --version
2.	Install Maven
a.	wget https://archive.apache.org/dist/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
b.	tar -xvzf apache-maven-3.9.5-bin.tar.gz
c.	mv ~/apache-maven-3.9.5 ~/maven
d.	echo 'export PATH=$PATH:~/maven/bin' >> ~/.bashrc
e.	source ~/.bashrc
f.	mvn --version
3.	Create new directory:
a.	mkdir dir_name
b.	cd dir_name
4.	Create Maven Project
a.	mvn archetype:generate -DgroupId=com.example -DartifactId=hello-world -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
b.	cd hello-world
c.	gedit pom.xml
d.	Add after dependencies
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-jar-plugin</artifactId>
<version>3.3.0</version>

<configuration>
<archive>
<manifest>
<mainClass>com.example.App</mainClass>
</manifest>
</archive>
</configuration>

</plugin>
</plugins>
</build>
5.	Build and run project
a.	mvn package
b.	java -cp target/hello-world-1.0-SNAPSHOT.jar com.example.App
output: Hello World
6.	Exit directory: cd..
7.	Install ansible following steps from program 7 if it is not installed
8.	Create files
a.	gedit hosts.ini
[local]
localhost ansible_connection=local
b.	gedit setup.yml
- name: Deploy Maven Artifact
  hosts: local

  tasks:

    - name: Create deploy directory
      file:
        path: /home/student/deploy
        state: directory

    - name: Copy JAR
      copy:
        src: /home/student/shemeema/hello-world/target/hello-world-1.0-SNAPSHOT.jar
        dest: /home/student/deploy/

    - name: Run JAR
      command: java -jar /home/student/deploy/hello-world-1.0-SNAPSHOT.jar
(REPLACE THE HIGHLIGHTED LINES ACCORDING TO YOUR DIRECTORY NAME)
9.	Download Jenkins
a.	wget https://get.jenkins.io/war-stable/latest/jenkins.war
b.	java -Xms512m -Xmx1024m -jar jenkins.war --httpPort=9090
c.	Login: Username: admin
get password using: cat ~/.jenkins/secrets/initialAdminPassword
d.	Create Pipeline and add the following script
pipeline {

agent any

environment {
PROJECT_DIR = "/home/student/shemeema/hello-world"
}

stages {

stage('Build') {
steps {

dir("${PROJECT_DIR}") {

sh '''
export PATH=$HOME/maven/bin:$PATH
mvn clean package
'''

}

}
}

stage('Verify') {
steps {
sh 'ls -l /home/student/shemeema/hello-world/target/'
}
}

stage('Deploy') {
steps {

sh '''
export LC_ALL=C.UTF-8
export LANG=C.UTF-8
export PATH=$HOME/.local/bin:$PATH

ansible-playbook -i /home/student/shemeema/hosts.ini /home/student/shemeema/setup.yml
'''

}
}

}

post {

success {
echo 'BUILD + DEPLOY SUCCESS ✅'
}

failure {
echo 'PIPELINE FAILED ❌'
}

}

}
(REPLACE THE HIGHLIGHTED LINES ACCORDING TO YOUR DIRECTORY NAME)
10.	Run the pipeline
