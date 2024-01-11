# Build/Setup a Jenkins Cluster

This creates a Jenkins Cluster consisting of:

- A Jenkins Master node, and
- 2 Jenkins Slave node

Docker images would be used for the purpose of the simulation.

## Jenkins Master & Slave Architecture

                  ─────────
                 │ Master
                 └─────────
     │────────────────│──────────────│
     │                │              │
    ──────      ────────            ──────
    │Slave      │Slave      ...     │Slave
    └──────     └───────            └──────

## Steps

### Initial Environment Prep

1.  Create Dockerfile
    In the project root directory, create a Dockerfile with the following content:

    These are base images for the Jenkins Master and Slave nodes, that would be pulled into the project root directory.

            FROM jenkins/jenkins

            FROM jenkins/jnlp-slave

2.  Login to docker:

            docker login
            Username: your_username
            Password:
            Login Succeeded

3.  Create the Docker Network:

    This custom Docker network to ensure that all containers (Jenkins Master and Jenkins Slave nodes) can communicate with each other using DNS. Enter the following in Terminal:

            docker network create myjenkinsnetwork

### Launch Jenkins Master Node

1.  Pull the master base image into the project root directory:

            pwd
            SetupJenkinsCluster
            docker pull jenkins/jenkins

2.  Create the Jenkins Master node container from the pulled image and attach it to the custom network:

            docker run -d --name jenkins-master \
              --network myjenkinsnetwork \
              -p 8080:8080 -p 50000:50000 \
              -v jenkins_home:/var/jenkins_home \
              jenkins/jenkins

3.  Retrieve Jenkins Master initial Admin password

            docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword

4.  Complete and verify Jenkins Master installation.
    - Visit http://localhost:8080
    - Enter the initial admin password, and
    - Follow the setup wizard to configure Jenkins.

### Launch Jenkins Slave Nodes

1.  Pull the slave base image into the project root directory:

            pwd
            SetupJenkinsCluster
            docker pull jenkins/jnlp-slave

2.  Create the Jenkins Slave nodes container from the pulled image and attach it to the custom network:

            docker run -d --name jenkins-node1 \
              --network myjenkinsnetwork \
              -e JENKINS_MASTER=http://jenkins-master:8080 \
              jenkins/jnlp-slave

            docker run -d --name jenkins-node2 \
              --network myjenkinsnetwork \
              -e JENKINS_MASTER=http://jenkins-master:8080 \
              jenkins/jnlp-slave

### Connect Slave Nodes to Jenkins Master

- Jenkins Master Dashboard => Manage Jenkins => System Configuration: Nodes => New Node
- Create two nodes with the name matching the script above: "jenkins-node1" and "jenkins-node2"
- Select: Permanent Agent
- Number of Executors: 2
- Remote root directory: /var/jenkins
- => Save

### Verify Jenkins Cluster

- Jenkins Master Dashboard => Manage Jenkins => System Configuration: Nodes

End.
