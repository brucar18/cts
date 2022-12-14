# cts

### CTS instructions  


####   
Backend

1.) Prepare environment  
Requirements for local building and deployment are:  
docker engine and git.  
In this example i am using debian bullseye latest.  
  
To install mentioned package run these commands:

```Bash
sudo apt install docker.io docker-compose git
```

2.) Build image.  
For backend i am using python code so you can use python-minimal docker image.  
Use dockerfile from repository to build image.

```Bash
git clone https://github.com/brucar18/cts.git
```

After cloning move into backend dir and run build command:

```Bash
docker build -t backend .
```

or you can use bash script added in folder.

```Bash
chmod +x buildApp.sh
./buildApp.sh
```

Now you have backend image.  
  
3.) Run and test app.  
To run and test your backend:

```Bash
docker run -it -p 80:80 backend
```

For testing you can use browser and check [http://localhost:80](http://localhost:80)/ping  
or you can use curl:

```Bash
aljaz@abMint:~# curl -I http://127.0.0.1/ping
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Sun, 28 Aug 2022 10:53:54 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 2
Connection: keep-alive
```

Backend is made with flask and it is running under uwsgi.  
Application port is **80**  
I have made post-commit and post merge hooks visible in .git/hooks   
For local setup every commit build new image.  
  
Cloud setup:  
For deploying images i would use docker registry. So every merge or commit into master should trigger jenkins webhook to build image and push it to registry ( step 1 ) I have made 2 jobs, backend-build and backend-build-deploy  
Step 2 is to trigger pull image and restart it on cloud solution. ( via api, webhooks .... )  
At the end of every build i run post task to start image and test curl output. 

```Bash
docker run -d -p 80:80 --name backend_test backend
curl http://backend_test/ping | grep OK 
#if response ok test done and stop conatainer else do not push to registry and send error.

```

For example i use jenkins webhooks to do this in my case.  
  
Example jenkins pipeline for image build.

```
pipeline {
environment {
dockerImage = 'backend'
# here we can add registry for pushing into own registry
agent any
stages {
stage('Building backend image') {
steps{
sh 'cd backend && docker build -t $dockerImage":$BUILD_NUMBER" .'
}
}
# tagging stage for registry and push
stage('Cleaning up') {
steps{
sh 'docker rmi $dockerImage":$BUILD_NUMBER"'
}
}
}
}
```

To have high availability system i would use haproxy ( or some other cloud load balacner ) on top of my backend servers ( see example config in git ). Haproxy config can be edited in mounted shared storage or via haproxy api.  
So my setup would look like this:  
On google cloud provider i would set up kuberenets cluster for managing apps. For cpu load i would use auto-scale options in workload area. To manage cointainers i would setup Google container registry ( gcr ) so i can use my own images in cloud and push it from jenkins build ...   
For networking i would use isolated network for containers and only expose haproxy or some loadbalancer port.
