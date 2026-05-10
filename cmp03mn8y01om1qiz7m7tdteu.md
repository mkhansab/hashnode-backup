---
title: "Jenkins Installation"
datePublished: 2026-05-10T18:21:03.923Z
cuid: cmp03mn8y01om1qiz7m7tdteu
slug: jenkins-installation
cover: https://cdn.hashnode.com/uploads/covers/6633c2252c01edc0085a6004/a6d607b8-333b-437e-bbab-4745ebc0767b.jpg

---

Steps to Install Jenkins:

1.  Clone the repo [https://github.com/mkhansab/install-jenkins-docker](https://github.com/mkhansab/install-jenkins-docker) and follow the 4 steps present at [README.md](https://github.com/mkhansab/install-jenkins-docker/blob/main/README.md) at this repo
    
2.  Opening [http://localhost:8080/](http://localhost:8080/) will ask for AdminPassword when setting up for the first time, password for this will be present at /var/jenkins\_home/secrets/initialAdminPassword  
    To view the password, Open Docker Desktop > my-jenkins Container > Exec and then `cat /var/jenkins_home/secrets/initialAdminPassword`
    
3.  Setting up for the first time?  
    A new page will open that asks for username and password, ensure to use a secure password as it'll be used to subsequent logins
    
4.  Hola, Jenkins is ready!