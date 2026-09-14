Damn Vulnerable Web Application (DVWA) is a PHP/MySQL web application that is damn vulnerable. Its main goals are to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications and aid teachers/students to teach/learn web application security in a class room environment.

---

## Setup:
1. After [[Docker]] setup is complete, pull DVWA:
```bash
# Start docker
sudo service docker start

# Run DVWA
sudo docker run -p 80:80 vulnerables/web-dvwa
```
2. The service should start on:
```website
http://localhost
```
3. The default credentials are:
	1. *Username:* `admin`
	2. *Password:* `password`
4. After logging in, click `Create / Reset Database"` at the bottom of the page.
5. Log in again.
6. Go to `DVWA Security` in the left menu and set the security level to `Low` 

---
## Useful commands:
1. List Container ID:
```shell
sudo docker ps
```
2. Start Container:
```shell
sudo docker start <container_id>
```
2. Stop Container:
```shell
sudo docker stop <container_id>
```

---
# Reverse SSH you DVWA:
```shell
ssh -R 8080:localhost:80 shrikant@taic
```
This makes your local port 80 (DVWA) available on the server at `localhost:8080`. Then in your agent config, set the DVWA target to:
```shell
http://127.0.0.1:8080
```

