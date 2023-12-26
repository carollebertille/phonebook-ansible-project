[![Build Status](http://34.204.91.44:8080/buildStatus/icon?job=phonebook)](http://34.204.91.44:8080/job/phonebook/)
# Deployment of the phonebook application

___________________________________________________

## 1. **Context**
   
   * Deployment of the phonebook application through the CI / CD
   
   * Implementation of the security aspect
   
   * Notification system

## 2. **Used tools**
   
       * Amazon EC2                                       
       * Docker
       * Jenkins 
       * Ansible                                         
       * Sonarqube                                        
       * Clair                                           
       * Dockerhub registry

## 3. **Infrastructure**
   

All servers are deployed on **AWS**.
We have on our infrastructure:

- **Ansible and Jenkins server:**
  
  This server is the main one. Thanks to Ansible, the application will be deployed automatically on the servers and thanks to Jenkins, we will be able to orchestrate this deployment.

- **Dockerhub registry:**

The goal here is to store and share our docker images. we used **Dockerhub**. 

- **build, preprod and production servers:**

They will be used respectively for the build of images, for deployments in test and production environments.

## 4. **Workflow**

It consists of two environments: recet and procuction
In order to fully understand this workfow, let's take the following scenario:

- Developer makes a modification to the code and pushes it on github

- Thanks to the webhook, the modification is received on the jenkins server and the build of the project can begin

- Syntax checks will be done (unit tests)

- We will have the build of the docker images and pushed on  **Dockerhub** registry. 

- Deployment in a test environment (pre-production) can begin by pulling images from Dockerhub register and a notification is sent to slack.

- The modification validated by the development team, a **PR** will be carried out in order to share the modification to the Operational (Ops)

- After validation by the whole team, the Ops manager can make the **merge request** in order to pass the modification on the master branch.

- Deployment in production environment will then be activated and a notification is sent to slack

- Once the application is deployed, a user will be able to connect and consume the application.

### Keywords

```
docker, docker compose, git, github, shared library, ansible, jenkins, AWS, Clair, Dockerhub registry, Slack, Pull Request, Merge Request
```

### References

* https://github.com/carollebertille/phonebook-dev source_code

* https://github.com/carollebertille/share-library in order to share and make reusable codes
