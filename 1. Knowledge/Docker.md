## What is it?
Docker will let you run an entire application along with all its dependencies inside a container. 

A container as a lightweight, self-contained box that has everything an application needs to run (the code, the dependencies, the operating system libraries) all packaged together.

In essence, a Docker container is similar to a [Domain Expansion](https://jujutsu-kaisen.fandom.com/wiki/Domain_Expansion) from the popular Japanese anime [Jujutsu Kaisen](https://en.wikipedia.org/wiki/Jujutsu_Kaisen)

DVWA (your vulnerable target website) is a whole web application with a database, a web server, PHP etc. Installing all of that manually on Kali would be painful and messy.

With Docker you just run one command and it spins up the entire thing instantly, already configured and ready to attack. When you're done you shut it down and it's gone — no cleanup needed.

---

# Useful Commands:

## Install:
```shell
sudo apt update && sudo apt install -y docker.io
```

## Start the Service:
```shell
sudo service docker start
```

## Verify:
```shell
docker --version
```

