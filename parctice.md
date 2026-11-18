DevOps Practicals – Step-by-Step Guide
Nov 17, 2026
·
@Dr. Sunita Yadav
Before you start (Windows + Docker Desktop)
Everything runs natively on Windows: commands in
PowerShell
(Windows Terminal),containers in
Docker Desktop
, Jenkins and Grafana as Windows services. Ubuntu is usedonly where Windows can't do the job:
1. Install the tools
(PowerShell as Administrator):
winget install --id Git.Git -ewinget install --id Microsoft.OpenJDK.21 -ewinget install --id Docker.DockerDesktop -ewinget install --id Microsoft.VisualStudioCode -e # editor, optional but handy
2. Install Maven
(no reliable winget package):
1.
Download the
Binary zip
of the latest Maven 3.9.x from
maven.apache.org/download.cgi
.
2.
Extract it to
C:\Program Files\Maven\
.
3.
Start menu → “Edit the system environment variables” →
Environment Variables
→under System variables select
Path
→
New
→ add the
bin
folder, e.g.
C:\ProgramFiles\Maven\apache-maven-3.9.11\bin
→ OK.
3. Verify
(open a
new
PowerShell window):
Practical
Runs on
Why
Q1–Q5, Q7–Q9
Windows (PowerShell + DockerDesktop)
All tools have Windows versions
Q6 Ansible
Ubuntu (WSL2) terminal
An Ansible control node can't run onWindows
Q10 NodeExporter
Docker container (Linux)
Node Exporter has no Windows build;Prometheus and Grafana stay native
DevOps Practicals – Step-by-Step Guide
Page 1 of 28
git --versionjava -versionmvn -versiondocker version # start Docker Desktop first
4. Docker Desktop settings
Settings → General
→
Use the WSL 2 based engine
stays ticked (the default).
ELK (Q9) needs about 4 GB. Create
C:\Users\<you>\.wslconfig
with the linesbelow, then run
wsl --shutdown
and restart Docker Desktop.
[wsl2]memory=6GB
PowerShell tips for this guide
Use
curl.exe
, not
curl
. In Windows PowerShell 5.1,
curl
is an alias for
Invoke-WebRequest
.
Chain commands with
;
(PowerShell 5.1 has no
&&
).
Create files with VS Code (
code filename
) or Notepad. For files with no extension,such as
Dockerfile
, run
notepad Dockerfile.
with a trailing dot so Notepaddoesn't add
.txt
.
Jenkins (Q2/Q3) and the nginx container (Q4) both use port 8080. Stop Jenkinsbefore Q4 (see Q4).
Q1. Version Control with Git
1. Initialize repo and configure user (4)
mkdir student-management; cd student-managementgit init -b maingit config user.name "Your Name"git config user.email "you@example.com"git config --list | Select-String user
2. Create files and make the initial commit (5)
DevOps Practicals – Step-by-Step Guide
Page 2 of 28
@'<!DOCTYPE html><html><head><title>Student Management</title><link rel="stylesheet" href="style.css"></head><body> <h1>Student Management System</h1> <script src="app.js"></script></body></html>'@ | Set-Content index.html'body { font-family: Arial; background: #f4f4f4; }' | Set-Content style.css"console.log('App loaded');" | Set-Content app.jsgit add .git commit -m "Initial commit"git log --oneline
Git may warn that “LF will be replaced by CRLF”. That's normal on Windows.
3. Branch feature-login with a login form (5)
git checkout -b feature-loginnotepad index.html
In
index.html
, add this inside
<body>
below the
<h1>
:
<form id="loginForm"> <input type="text" placeholder="Username" required> <input type="password" placeholder="Password" required> <button type="submit">Login</button></form>
git add index.htmlgit commit -m "Add login form"
4. Switch to main, modify app.js, commit (4)
DevOps Practicals – Step-by-Step Guide
Page 3 of 28
git checkout mainAdd-Content app.js "function greet(){ alert('Welcome Student'); }"git add app.jsgit commit -m "Add greet function in app.js"
5. Merge feature-login into main (4)
git merge feature-login -m "Merge feature-login into main"git log --oneline
The branches touched different files, so this merges without conflict.
6. Create and resolve a merge conflict (8)
Edit the same line (the
<h1>
) differently in two branches:
# On a new branchgit checkout -b feature-title(Get-Content index.html) -replace '<h1>.*</h1>', '<h1>Student Portal - Feature</h1>' | Set-Content index.htmlgit commit -am "Change title in feature-title"# Same line on maingit checkout main(Get-Content index.html) -replace '<h1>.*</h1>', '<h1>Student Portal - Main</h1>' | Set-Content index.htmlgit commit -am "Change title in main"# Merge -> conflictgit merge feature-titlegit status # shows: both modified: index.html
Open
index.html
. Git shows:
<<<<<<< HEAD <h1>Student Portal - Main</h1>======= <h1>Student Portal - Feature</h1>>>>>>>> feature-title
Open it with
notepad index.html
(or VS Code, which shows
Accept Current / Incoming
buttons). Keep the version you want, delete the three marker lines, save, then:
DevOps Practicals – Step-by-Step Guide
Page 4 of 28
git add index.htmlgit commit -m "Resolve merge conflict in title"git log --graph --oneline --all
The graph output shows the branches splitting and joining at the merge commits.
Q2. Setting Up Jenkins + CI Job
These steps use the Jenkins already installed on Windows (it runs as a Windows service).Build steps use Windows commands (
bat
), not Linux
sh
.
1. Install and start Jenkins, create admin user, install suggested plugins (6)
Jenkins is already installed, so check that it's running. Open
Services
(
Win + R
→
services.msc
) →
Jenkins
should be
Running
. Or in PowerShell as Administrator:
Get-Service jenkinsStart-Service jenkins # if it is stopped# Initial admin password (first-time setup only), one of:Get-Content "C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword"Get-Content "C:\Program Files\Jenkins\secrets\initialAdminPassword"
If you already finished the setup wizard earlier, just log in at
http://localhost:8080
andskip the list below. For the practical record, screenshot the dashboard and
ManageJenkins → Plugins → Installed plugins
.
1.
Open
http://localhost:8080
and paste the initial admin password.
2.
Click
Install suggested plugins
and wait for it to finish.
3.
Fill in the
Create First Admin User
form (username, password, name, email) →
Saveand Continue
→
Start using Jenkins
.
Configure Maven and Git in Jenkins (needed on Windows)
1.
Install
Git for Windows
if you don't have it (
git-scm.com
).
2.
Manage Jenkins → Tools
:
Git installations
→ Path to Git executable:
C:\Program Files\Git\bin\git.exe
(only if plain
git
doesn't work).
Maven installations
→
Add Maven
→ Name:
Maven3
→ tick
Install automatically
(pick the latest 3.9.x) →
Save
.
DevOps Practicals – Step-by-Step Guide
Page 5 of 28
3.
Jenkins uses its own Java (17 or 21) to run the build, so you don't need a separate JDKsetting.
Prepare a sample Java Maven project (used in Q2 and Q3)
Create it in PowerShell and push it to GitHub; Jenkins then clones it from GitHub. InPowerShell, every
-D...
argument must be in quotes.
mvn archetype:generate "-DgroupId=com.demo" "-DartifactId=java-demo" ` "-DarchetypeArtifactId=maven-archetype-quickstart" "-DarchetypeVersion=1.4" ` "-DinteractiveMode=false"cd java-demo# The archetype targets Java 1.7, which JDK 17/21 can't compile. Set it to 17:(Get-Content pom.xml) -replace '>1.7<', '>17<' | Set-Content pom.xmlSelect-String compiler pom.xml # both lines should now show 17mvn clean package # confirm it builds locally firstgit init -b main; git add .; git commit -m "Sample Maven project"# Create an empty repo java-demo on GitHub, then:git remote add origin https://github.com/<your-username>/java-demo.gitgit push -u origin main
2. Create a Freestyle job named java-build-job (4)
1.
Dashboard →
New Item
.
2.
Name:
java-build-job
→ select
Freestyle project
→
OK
.
3. Connect the job to the Java project (6)
1.
Source Code Management
→ select
Git
.
2.
Repository URL:
https://github.com/<your-username>/java-demo.git
3.
Branch Specifier:
*/main
(add credentials only if the repo is private).
4. Configure triggers and build steps (7)
Build Triggers
→ tick
Poll SCM
→ Schedule:
H/2 * * * *
(checks Git every 2minutes).
Alternative: tick
GitHub hook trigger for GITScm polling
and add a GitHub webhookto
http://<public-url>/github-webhook/
. Your Windows PC isn't reachable fromGitHub, so this needs a tunnel such as ngrok. Poll SCM is simpler.
DevOps Practicals – Step-by-Step Guide
Page 6 of 28
Build Steps
→
Add build step
→
Invoke top-level Maven targets
→ Maven Version:
Maven3
→ Goals:
clean compile package
.
Alternative:
Execute Windows batch command
(works only if Maven is on the WindowsPATH and the Jenkins service was restarted afterwards):
mvn clean compilemvn package
Optional
Post-build Actions
→
Archive the artifacts
→
target/*.jar
.
Click
Save
.
5. Run the build and show Console Output (7)
1.
Click
Build Now
.
2.
Open the build (#1) in
Build History
→
Console Output
. It should end with
BUILDSUCCESS
and
Finished: SUCCESS
.
Common failures and fixes:
Re-run
Build Now
after the fix and capture the successful console output.
Q3. CI/CD Pipeline with Jenkins and Maven
Use the same
java-demo
repo from Q2.
Error in console
Fix
'mvn' is not recognized as aninternal or external command
Use
Invoke top-level Maven targets
with
Maven3
, configured under Manage Jenkins →Tools
Cannot run program "git.exe"
Install Git for Windows, or set the git.exe pathunder Manage Jenkins → Tools
Couldn't find any revision to build
Branch is
main
, not
master
: set BranchSpecifier to
*/main
Source option 7 is no longersupported
Set
maven.compiler.source
/
target
in
pom.xml
to
17
(the
sed
step above)
Test failure
Fix the test, or temporarily add
-DskipTests
to the goals
DevOps Practicals – Step-by-Step Guide
Page 7 of 28
1. Create a Pipeline job named cicd-pipeline (4)
1.
Dashboard →
New Item
→ name
cicd-pipeline
→
Pipeline
→
OK
.
2.
Under
Pipeline
, set Definition =
Pipeline script from SCM
, SCM =
Git
, Repository URL= your repo, Branch =
*/main
, Script Path =
Jenkinsfile
.
3.
Under
Build Triggers
, tick
Poll SCM
with
H/2 * * * *
(needed for part 4) →
Save
.
2. Write the Jenkinsfile (16)
Create
Jenkinsfile
(no extension) in the root of the
java-demo
folder, e.g.
codeJenkinsfile
or
notepad Jenkinsfile.
This version is for Jenkins on Windows (
bat
steps,
Maven3
from Manage Jenkins →Tools):
pipeline { agent any tools { maven 'Maven3' } environment { DEPLOY_DIR = 'C:\\deploy' } stages { stage('Checkout') { steps { checkout scm } } stage('Build') { steps { bat 'mvn clean compile' } } stage('Test') { steps { bat 'mvn test' } post { always {
DevOps Practicals – Step-by-Step Guide
Page 8 of 28
junit 'target/surefire-reports/*.xml' } } } stage('Package') { steps { bat 'mvn package -DskipTests' archiveArtifacts artifacts: 'target/*.jar', fingerprint: true } } stage('Deploy') { steps { bat ''' if not exist "%DEPLOY_DIR%" mkdir "%DEPLOY_DIR%" copy /Y target\\*.jar "%DEPLOY_DIR%\\" dir "%DEPLOY_DIR%" ''' } } } post { success { echo 'Pipeline completed successfully!' } failure { echo 'Pipeline failed.' } }}
The double backslashes are Groovy escaping; Windows receives
C:\deploy
and
target\*.jar
. After a run, open
C:\deploy
in File Explorer to show the deployed
.jar
.On Linux Jenkins, swap each
bat
for
sh
and use
cp
/
mkdir -p
instead.
For a Tomcat deploy instead: set packaging to
war
in
pom.xml
, and in the Deploy stagecopy the war into Tomcat's webapps folder, e.g.
copy /Y target\\*.war "C:\\ProgramFiles\\Apache Software Foundation\\Tomcat 10.1\\webapps\\"
. Tomcat must berunning on a port other than 8080, since Jenkins already uses it.
Push it:
git add Jenkinsfilegit commit -m "Add Jenkinsfile"git push
3. Run the pipeline and show Stage View (5)
DevOps Practicals – Step-by-Step Guide
Page 9 of 28
1.
Open
cicd-pipeline
→
Build Now
.
2.
The job page shows the
Stage View
grid: Checkout → Build → Test → Package →Deploy, each green with its time.
3.
If no Stage View appears, install
Pipeline: Stage View
(or
Pipeline Graph View
) fromManage Jenkins → Plugins.
4. Demonstrate automatic trigger on code change (5)
# Make any change in the repoAdd-Content src\main\java\com\demo\App.java "// trigger build"git commit -am "Trigger pipeline"git push
Within 2 minutes a new build starts on its own. Open the build → its cause reads
Startedby an SCM change
. Use
Git Polling Log
on the job page as extra proof. With a GitHubwebhook instead of polling, the build starts within seconds.
Q4. Docker Basics
First stop Windows Jenkins, which also uses port 8080. In PowerShell as Administrator,run
Stop-Service jenkins
(and
Start-Service jenkins
later). You can also see thecontainers and images in the Docker Desktop window.
1. Pull nginx and run it on port 8080 (6)
docker --versiondocker pull nginxdocker run -d --name my-nginx -p 8080:80 nginxcurl.exe http://localhost:8080
Open
http://localhost:8080
in a browser → you see the
Welcome to nginx!
page.
2. Create a simple web app (5)
DevOps Practicals – Step-by-Step Guide
Page 10 of 28
mkdir hello-docker; cd hello-docker@'<!DOCTYPE html><html><head><title>Docker App</title></head><body><h1>Hello Docker</h1></body></html>'@ | Set-Content index.html
3. Write the Dockerfile (8)
Create a file named
Dockerfile
(no extension) in the same folder:
notepadDockerfile.
(the trailing dot stops Notepad adding
.txt
), or
code Dockerfile
.
# Base imageFROM nginx:alpine# MetadataLABEL maintainer="you@example.com"# Replace the default nginx page with our appCOPY index.html /usr/share/nginx/html/index.html# nginx listens on port 80EXPOSE 80# Run nginx in the foregroundCMD ["nginx", "-g", "daemon off;"]
4. Build as my-web-app:v1 and run on port 8081 (6)
docker build -t my-web-app:v1 .docker run -d --name web-app -p 8081:80 my-web-app:v1curl.exe http://localhost:8081
Open
http://localhost:8081
→ the page shows
Hello Docker
.
5. Management commands (5)
DevOps Practicals – Step-by-Step Guide
Page 11 of 28
docker ps # running containersdocker ps -a # all containers, including stoppeddocker images # local images (nginx, my-web-app:v1)docker logs web-app # access logs of the containerdocker logs -f my-nginx # follow logs live (Ctrl+C to exit)docker stop web-app my-nginxdocker rm web-app my-nginxdocker ps -a # confirm they are gone
Q5. Docker Compose – Multi-Container App
This uses PHP + Apache for the web app (it listens on port 80, matching
8000:80
).
1. Create the project (5)
mkdir compose-app; cd compose-appmkdir src, dbcode . # create the files below in VS Code (or use Notepad)
Folder layout:
compose-app/├── docker-compose.yml├── Dockerfile├── src/index.php└── db/init.sql
Dockerfile
:
FROM php:8.2-apacheRUN docker-php-ext-install mysqliCOPY src/ /var/www/html/EXPOSE 80
db/init.sql
(runs automatically on the first start of MySQL):
DevOps Practicals – Step-by-Step Guide
Page 12 of 28
CREATE TABLE IF NOT EXISTS students ( id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50), course VARCHAR(50));INSERT INTO students (name, course) VALUES ('Asha', 'MSc CS'), ('Rahul', 'MSc CS'), ('Neha', 'MCA');
src/index.php
:
<?php$conn = new mysqli('db', 'root', 'rootpass', 'studentdb');if ($conn->connect_error) { die('DB connection failed: ' . $conn->connect_error);}echo "<h2>Connected to MySQL successfully</h2>";$result = $conn->query('SELECT * FROM students');echo "<table border='1'><tr><th>ID</th><th>Name</th><th>Course</th></tr>";while ($row = $result->fetch_assoc()) { echo "<tr><td>{$row['id']}</td><td>{$row['name']}</td><td>{$row['course']}</td></tr>";}echo "</table>";$conn->close();?>
2. Write docker-compose.yml (12)
services: web: build: . ports: - "8000:80" depends_on: db: condition: service_healthy db: image: mysql:8.0 environment: MYSQL_ROOT_PASSWORD: rootpass MYSQL_DATABASE: studentdb volumes:
DevOps Practicals – Step-by-Step Guide
Page 13 of 28
- db_data:/var/lib/mysql - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql healthcheck: test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-prootpass"] interval: 10s retries: 10volumes: db_data:
The healthcheck makes
web
wait until MySQL is actually ready, not just started.
3. Start and verify both containers (5)
docker compose up -d --builddocker compose ps # both web and db show "running" / "healthy"docker ps
4. Verify the app reads from the database (5)
Open
http://localhost:8000
→ the page shows
Connected to MySQL successfully
and the students table. Check from the DB side too:
docker compose exec db mysql -uroot -prootpass -e "SELECT * FROM studentdb.students;"
5. Scale down, logs, bring down with volumes (3)
docker compose up -d --scale web=0 # scale web down to 0 replicasdocker compose psdocker compose up -d --scale web=1 # bring it backdocker compose logs # all servicesdocker compose logs -f web # follow one servicedocker compose down -v # stop, remove containers, network AND the db_data volumedocker volume ls # db_data is gone
web
publishes a fixed host port (8000), so it can't scale above 1 replica on one machine.
DevOps Practicals – Step-by-Step Guide
Page 14 of 28
Q6. Introduction to Ansible
This is the one practical that needs Ubuntu.
Ansible's control node only runs on Linux, soit runs in Ubuntu on WSL2. The
2 managed nodes
are two Ubuntu containers in DockerDesktop with SSH:
web1
(SSH port 2221, web 8091) and
web2
(SSH 2222, web 8092).
Set up Ubuntu (once)
1.
PowerShell as Administrator:
wsl --install -d Ubuntu-24.04
. Restart if asked,then open
Ubuntu
from the Start menu and create a Linux username and password.
2.
Docker Desktop →
Settings → Resources → WSL integration
→ turn on
Ubuntu-24.04
→
Apply & restart
. Now
docker
works inside Ubuntu too.
3.
Run
every Q6 command below in the Ubuntu terminal
, not PowerShell.
0. Create the two managed nodes
mkdir -p ~/ansible-lab/node && cd ~/ansible-labssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""cp ~/.ssh/id_ed25519.pub node/
node/Dockerfile
:
FROM ubuntu:22.04RUN apt-get update && apt-get install -y openssh-server python3 sudo \ && useradd -m -s /bin/bash ansible \ && echo 'ansible ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/ansible \ && mkdir -p /var/run/sshdCOPY id_ed25519.pub /home/ansible/.ssh/authorized_keysRUN chown -R ansible:ansible /home/ansible/.ssh \ && chmod 700 /home/ansible/.ssh && chmod 600 /home/ansible/.ssh/authorized_keysEXPOSE 22 80CMD ["/usr/sbin/sshd", "-D"]
docker build -t ansible-node ./nodedocker run -d --name web1 -p 2221:22 -p 8091:80 ansible-nodedocker run -d --name web2 -p 2222:22 -p 8092:80 ansible-nodedocker ps
1. Install Ansible on the control node (4)
DevOps Practicals – Step-by-Step Guide
Page 15 of 28
sudo apt updatesudo apt install -y software-properties-commonsudo add-apt-repository --yes --update ppa:ansible/ansiblesudo apt install -y ansibleansible --version
Test SSH to both nodes (the key was built into the image, so no password is needed):
ssh -p 2221 -o StrictHostKeyChecking=no ansible@localhost hostnamessh -p 2222 -o StrictHostKeyChecking=no ansible@localhost hostname
The
ansible
user in the containers already has passwordless sudo.
2. Inventory and connectivity test (5)
cd ~/ansible-lab
inventory.ini
:
[webservers]web1 ansible_host=127.0.0.1 ansible_port=2221web2 ansible_host=127.0.0.1 ansible_port=2222[webservers:vars]ansible_user=ansibleansible_python_interpreter=/usr/bin/python3
ansible.cfg
(so you don't need
-i
every time):
[defaults]inventory = inventory.inihost_key_checking = False
ansible all -m ping
Expected:
web1 | SUCCESS => {"ping": "pong"}
and the same for
web2
.
3. Playbook install-packages.yml (12)
First create
files/index.html
:
mkdir filesecho "<h1>Apache deployed by Ansible</h1>" > files/index.html
DevOps Practicals – Step-by-Step Guide
Page 16 of 28
install-packages.yml
:
---- name: Configure web servers hosts: webservers become: yes tasks: - name: Update apt cache apt: update_cache: yes cache_valid_time: 3600 - name: Install required packages apt: name: - apache2 - curl - git - vim state: present - name: Ensure apache2 is started and enabled service: name: apache2 state: started enabled: yes - name: Copy custom index.html copy: src: files/index.html dest: /var/www/html/index.html owner: www-data group: www-data mode: '0644'
cache_valid_time
stops the cache update from counting as a change on every run,which matters for the idempotency check.
4. Run the playbook (5)
ansible-playbook install-packages.yml --syntax-checkansible-playbook install-packages.yml
DevOps Practicals – Step-by-Step Guide
Page 17 of 28
The
PLAY RECAP
shows
ok=5 changed=4
(or similar) for each host with
failed=0
.
5. Verify Apache and prove idempotency (4)
Open
http://localhost:8091
(web1) and
http://localhost:8092
(web2) in yourWindows browser → the custom page appears. Or:
ansible all -a "curl -slocalhost"
.
Run the playbook again:
ansible-playbook install-packages.yml
The recap now shows
changed=0
for both hosts: Ansible saw everything was already inthe desired state and changed nothing.
Q7. Kubernetes Basics
These steps use the
Kubernetes built into Docker Desktop
(a one-node cluster).
kubectl
comes with Docker Desktop and works in PowerShell.
Setup (once)
1.
Docker Desktop →
Settings → Kubernetes
→ tick
Enable Kubernetes
.
2.
If asked for a cluster type, choose
Kubeadm
(single node; NodePorts open on
localhost
).
3.
Apply & restart
and wait until the Kubernetes icon at the bottom left turns green (afew minutes).
kubectl config use-context docker-desktopkubectl version
1. Check cluster status (3)
kubectl cluster-infokubectl get nodes # STATUS should be Ready
2. Create a Pod imperatively (5)
kubectl run my-pod --image=nginxkubectl get pods # my-pod -> Runningkubectl get pods -o wide # shows pod IP and node
3. Deployment with 3 replicas from YAML (8)
DevOps Practicals – Step-by-Step Guide
Page 18 of 28
deployment.yaml
:
apiVersion: apps/v1kind: Deploymentmetadata: name: web-deployment labels: app: webspec: replicas: 3 selector: matchLabels: app: web template: metadata: labels: app: web spec: containers: - name: nginx image: nginx ports: - containerPort: 80
kubectl apply -f deployment.yamlkubectl get deploymentskubectl get pods -l app=web # 3 pods Running
4. NodePort Service and browser access (8)
service.yaml
:
DevOps Practicals – Step-by-Step Guide
Page 19 of 28
apiVersion: v1kind: Servicemetadata: name: web-servicespec: type: NodePort selector: app: web ports: - port: 80 targetPort: 80 nodePort: 30080
kubectl apply -f service.yaml# (imperative alternative: kubectl expose deployment web-deployment --type=NodePort --port=80)kubectl get svc web-service # PORT(S) shows 80:30080/TCPkubectl get nodes -o wide # INTERNAL-IP = the Node IP
On Docker Desktop the node is reached through
localhost
, so open
http://localhost:30080
in your Windows browser → nginx welcome page. For therecord, write it as NodeIP:NodePort, e.g.
<INTERNAL-IP>:30080
, and note that DockerDesktop maps it to localhost.
5. Required outputs (6)
kubectl get podskubectl get deploymentskubectl get serviceskubectl describe pod my-pod # or any pod name from 'kubectl get pods'
describe pod
shows the node, IP, container image, state, and the Events list (Scheduled,Pulling, Pulled, Created, Started).
Q8. Kubernetes Scaling, Rolling Updates & Rollbacks
Setup: create my-app (if not already running)
DevOps Practicals – Step-by-Step Guide
Page 20 of 28
kubectl create deployment my-app --image=nginx:1.25 --replicas=2kubectl annotate deployment/my-app kubernetes.io/change-cause="Initial deploy nginx:1.25"kubectl get deployment my-app
The container inside is named
nginx
(from the image name);
set image
uses that name.
1. Scale from 2 to 5 replicas (5)
kubectl scale deployment/my-app --replicas=5kubectl get deployment my-app # READY 5/5kubectl get pods -l app=my-app # 5 pods Running
2. Rolling update to nginx:1.27 (8)
kubectl set image deployment/my-app nginx=nginx:1.27kubectl annotate deployment/my-app kubernetes.io/change-cause="Update to nginx:1.27" --overwrite
Kubernetes replaces pods gradually (by default max 25% unavailable / 25% surge), so theapp stays up during the update.
3. Monitor the rollout (5)
Run these right after
set image
(use a second terminal for
-w
):
kubectl rollout status deployment/my-app# Waiting for deployment "my-app" rollout to finish: 2 out of 5 new replicas have been updated...# deployment "my-app" successfully rolled outkubectl get pods -w # watch old pods Terminating and new ones Running; Ctrl+C to stop
4. Rollout history (4)
kubectl rollout history deployment/my-app# REVISION CHANGE-CAUSE# 1 Initial deploy nginx:1.25# 2 Update to nginx:1.27kubectl rollout history deployment/my-app --revision=2 # details of one revision
DevOps Practicals – Step-by-Step Guide
Page 21 of 28
5. Roll back and verify the image (8)
kubectl rollout undo deployment/my-app# (or a specific one: kubectl rollout undo deployment/my-app --to-revision=1)kubectl rollout status deployment/my-appkubectl describe deployment my-app | Select-String Image# Image: nginx:1.25kubectl get deployment my-app -o jsonpath='{.spec.template.spec.containers[0].image}'kubectl rollout history deployment/my-app # revision 1 becomes revision 3
Q9. Centralized Logging with ELK Stack
Runs all three with Docker Compose, security turned off for a lab setup. Keep all three onthe same version. Needs about 4 GB free RAM.
1. Install and start Elasticsearch, Logstash, Kibana (8)
mkdir elk; cd elkmkdir logstash\pipeline, logs# If Elasticsearch later exits with a "vm.max_map_count" error, run once:# wsl -d docker-desktop sysctl -w vm.max_map_count=262144
docker-compose.yml
:
services: elasticsearch: image: docker.elastic.co/elasticsearch/elasticsearch:8.17.0 environment: - discovery.type=single-node - xpack.security.enabled=false - ES_JAVA_OPTS=-Xms1g -Xmx1g ports: - "9200:9200" volumes: - es_data:/usr/share/elasticsearch/data logstash: image: docker.elastic.co/logstash/logstash:8.17.0 volumes:
DevOps Practicals – Step-by-Step Guide
Page 22 of 28
- ./logstash/pipeline:/usr/share/logstash/pipeline - ./logs:/logs ports: - "5044:5044" depends_on: - elasticsearch kibana: image: docker.elastic.co/kibana/kibana:8.17.0 environment: - ELASTICSEARCH_HOSTS=http://elasticsearch:9200 ports: - "5601:5601" depends_on: - elasticsearchvolumes: es_data:
Write
logstash.conf
(part 2) before starting, then:
docker compose up -ddocker compose pscurl.exe http://localhost:9200 # JSON with cluster name and versioncurl.exe "http://localhost:9200/_cluster/health?pretty"
2. Logstash pipeline (8)
logstash/pipeline/logstash.conf
:
input { file { path => "/logs/access.log" start_position => "beginning" sincedb_path => "/dev/null" } beats { port => 5044 }}filter {
DevOps Practicals – Step-by-Step Guide
Page 23 of 28
grok { match => { "message" => "%{COMBINEDAPACHELOG}" } ecs_compatibility => disabled } date { match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ] } mutate { convert => { "response" => "integer" "bytes" => "integer" } }}output { elasticsearch { hosts => ["http://elasticsearch:9200"] index => "logs-%{+YYYY.MM.dd}" } stdout { codec => rubydebug }}
Input
: reads the log file (and also accepts Filebeat on 5044).
Filter
:
grok
splits each Apache line into fields (clientip, verb, request, response,bytes);
date
sets the event time;
mutate
makes status a number.
Output
: sends to Elasticsearch, one index per day.
3. Generate sample Apache logs (4)
gen_logs.ps1
(in the
elk
folder):
DevOps Practicals – Step-by-Step Guide
Page 24 of 28
$pages = "/index.html", "/login", "/api/students", "/about", "/contact"$inv = [Globalization.CultureInfo]::InvariantCulture$zone = (Get-Date).ToString("zzz").Replace(":", "") # e.g. +0530$lines = 1..200 | ForEach-Object { $ip = "192.168.1.$(Get-Random -Minimum 1 -Maximum 51)" $page = $pages | Get-Random $status = if ((Get-Random -Maximum 5) -eq 0) { 500 } else { 200 } $ts = (Get-Date).ToString("dd/MMM/yyyy:HH:mm:ss", $inv) + " $zone" "$ip - - [$ts] `"GET $page HTTP/1.1`" $status $(Get-Random -Maximum 5000) `"-`" `"Mozilla/5.0`""}$lines | Add-Content -Path logs\access.log -Encoding ascii"Generated $((Get-Content logs\access.log).Count) lines"
powershell -ExecutionPolicy Bypass -File .\gen_logs.ps1docker compose logs -f logstash # see parsed events (Ctrl+C to exit)curl.exe "http://localhost:9200/_cat/indices?v" # a logs-YYYY.MM.dd index appears
About 1 in 5 lines are status 500, the rest 200.
4. Kibana data view for logs-* (5)
1.
Open
http://localhost:5601
(wait 1–2 minutes after start).
2.
Menu →
Stack Management
→
Data Views
→
Create data view
.
3.
Name:
logs
, Index pattern:
logs-*
, Timestamp field:
@timestamp
→
Save data viewto Kibana
.
5. Discover search and visualization (5)
In
Discover
(data view
logs
, time range “Last 24 hours”):
Total log count
: the hit count at the top, e.g. “200 hits”.
Errors only
: query
response : 500
→ about 40 hits.
Success only
: query
response : 200
→ about 160 hits.
Visualization: Menu →
Visualize Library
→
Create visualization
→
Lens
:
1.
Chart type
Pie
(or Bar).
2.
Drag
response
into
Slice by
and keep
Count of records
as the metric → one slice for200, one for 500.
DevOps Practicals – Step-by-Step Guide
Page 25 of 28
3.
Add a
Metric
visualization with
Count of records
for the total.
4.
Save
both and, optionally, add them to a dashboard.
Q10. Monitoring with Prometheus and Grafana
Prometheus and Grafana run natively on Windows. Node Exporter has no Windows build,so it runs as a Docker container on port 9100 and reports the Docker Desktop Linux VM'sCPU, memory, disk and network. Dashboard 1860 is built for Node Exporter, so stick with itrather than
windows_exporter
. Versions are examples; take the latest from
prometheus.io/download
if a link 404s.
1. Install and start Prometheus (5)
cd $HOME$v = "3.5.0"Invoke-WebRequest "https://github.com/prometheus/prometheus/releases/download/v$v/prometheus-$v.windows-amd64.zip" -OutFile prometheus.zipExpand-Archive prometheus.zip -DestinationPath C:\Rename-Item "C:\prometheus-$v.windows-amd64" C:\prometheuscd C:\prometheus.\prometheus.exe --version
Edit
prometheus.yml
(part 2), then start Prometheus.
Keep this window open
, becauseclosing it stops Prometheus:
.\prometheus.exe --config.file=prometheus.yml
Allow it if Windows Firewall asks. In a second PowerShell window,
curl.exehttp://localhost:9090/-/ready
should print “Prometheus Server is Ready.” Open
http://localhost:9090
→ Prometheus UI.
2. Configure prometheus.yml (7)
Replace the contents of
C:\prometheus\prometheus.yml
(
notepadC:\prometheus\prometheus.yml
):
DevOps Practicals – Step-by-Step Guide
Page 26 of 28
global: scrape_interval: 15s evaluation_interval: 15sscrape_configs: - job_name: "prometheus" static_configs: - targets: ["localhost:9090"] - job_name: "node_exporter" static_configs: - targets: ["localhost:9100"]
Check the file, then restart Prometheus: press
Ctrl+C
in its window and run the startcommand again.
cd C:\prometheus.\promtool.exe check config prometheus.yml
3. Run Node Exporter and check targets (5)
docker run -d --name node-exporter -p 9100:9100 --pid=host ` -v /:/host:ro,rslave quay.io/prometheus/node-exporter:latest --path.rootfs=/hostdocker pscurl.exe -s http://localhost:9100/metrics | Select-Object -First 10
In the Prometheus UI →
Status → Target health
(called
Targets
in older versions): both
prometheus
and
node_exporter
show
UP
. Try a query in the Query tab, e.g.
node_memory_MemAvailable_bytes
or
up
.
4. Install Grafana and add Prometheus as a data source (5)
1.
Download the
Windows installer (.msi)
of Grafana OSS from
grafana.com/grafana/download
and run it. It installs Grafana as a Windows service onport 3000.
2.
Check it's running:
Get-Service grafana
in PowerShell.
3.
Open
http://localhost:3000
, log in with
admin
/
admin
, set a new password.
4.
Connections → Data sources → Add data source → Prometheus
.
5.
Prometheus server URL:
http://localhost:9090
→
Save & test
→ “Successfullyqueried the Prometheus API.”
DevOps Practicals – Step-by-Step Guide
Page 27 of 28
5. Import dashboard 1860 – Node Exporter Full (8)
1.
Dashboards → New → Import
.
2.
Enter
1860
→
Load
.
3.
Select your Prometheus data source →
Import
.
4.
At the top, pick Job =
node_exporter
and Host =
localhost:9100
; set time range to
Last 15 minutes
.
The dashboard shows live panels for
CPU Busy / CPU Basic
,
RAM Used / Memory Basic
,
Disk Space Used / Disk IOps
, and
Network Traffic
. To make the graphs move, start a CPUload in the Docker VM:
docker run -d --name cpuload alpine sh -c "while true; do :; done"# watch CPU Busy rise, then stop it:docker rm -f cpuload
DevOps Practicals – Step-by-Step Guide
Page 28 of 28
