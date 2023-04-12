# Deploying a sample PHP To-Do 3 Tier Application on Kubernetes Cluster

#### In this project we are going to deploy a sample php to-do application to Kubernetes cluster and the architecture is going to 3 tier where we are using MySql as Database and phpMyAdmin as Database Management and php for developing the application.

![Screenshot (231)](https://user-images.githubusercontent.com/129657174/231373502-48477aec-088b-4a06-a76a-a3dcca55f195.png)

**Prerequisites:** Git, GitHub, Docker, Kubernetes cluster, MySql

**Requirements:**

![Project 4](https://user-images.githubusercontent.com/129657174/231373516-b2456083-d04a-49d9-8470-b08819204c53.png)

### Creating Kubernetes cluster using Kubeadm
-	Go to AWS Management Console 
-	EC2 -> Launch Instance
-	Launch two EC2 instances with Ubuntu image and t2.medium instance type, one is for master and other for  worker node

![Screenshot (233)](https://user-images.githubusercontent.com/129657174/231374363-399413a3-f95c-41e3-a4fd-c19efaa8b0f4.png)

### What is Kubeadm?

Kubeadm is a tool built to provide kubeadm init and kubeadm join as best-practice "fast paths" for creating Kubernetes clusters.

kubeadm performs the actions necessary to get a minimum viable cluster up and running. By design, it cares only about bootstrapping, not about provisioning machines. Likewise, installing various nice-to-have addons, like the Kubernetes Dashboard, monitoring solutions, and cloud-specific addons, is not in scope.

Instead, we expect higher-level and more tailored tooling to be built on top of kubeadm, and ideally, using kubeadm as the basis of all deployments will make it easier to create conformant clusters.

#### Installing kubeadm and neccesary packages on master and worker nodes. For reference check the kubeadm_master_install.sh and kubeadm_node_install.sh shell scripts.

Now connect to master node

```shell
   sudo apt update -y
   sudo apt install git -y #By default git is installed on Ubuntu 22.04
   git clone https://github.com/iamsaikishore/Project4---Deploying-a-sample-PHP-To-Do-3-Tier-Application-on-Kubernetes-Cluster.git
   cd Project4---Deploying-a-sample-PHP-To-Do-3-Tier-Application-on-Kubernetes-Cluster
   ./kubeadm_master_install.sh
```

Now connect to worker node

```shell
   sudo apt install git -y
   git clone https://github.com/iamsaikishore/Project4---Deploying-a-sample-PHP-To-Do-3-Tier-Application-on-Kubernetes-Cluster.git
   cd Project4---Deploying-a-sample-PHP-To-Do-3-Tier-Application-on-Kubernetes-Cluster
   ./kubeadm_node_install.sh
```

Now in the master node execute the below command

```kubeadm init --kubernetes-version=${KUBE_VERSION}```
   
Now copy the token and execute in the worker node





### What is MySQL?

MySQL is the world's most popular open source database. With its proven performance, reliability and ease-of-use, MySQL has become the leading database choice for web-based applications, covering the entire range from personal projects and websites, via e-commerce and information services, all the way to high profile web properties including Facebook, Twitter, YouTube, Yahoo! and many more.

For more information and related downloads for MySQL Server and other MySQL products, please visit [www.mysql.com](https://www.mysql.com/)
 
**MYSQL_DATABASE** This variable is optional and allows you to specify the name of a database to be created on image startup. If a user/password was supplied then that user will be granted superuser access ([corresponding to GRANT ALL](https://dev.mysql.com/doc/refman/8.0/en/adding-users.html)) to this database.

**MYSQL_ROOT_PASSWORD** This variable is mandatory and specifies the password that will be set for the MySQL root superuser account. In the below example, it was set to rootpassword.

Now will create a mysql pod, we require some neccesary configurations for that we create configmap for database name and secret for username and passsword

```shell
   kubectl create cm db-config –from-literal=MYSQL_DATABASE=sqldb
   kubectl create secret generic db-secret –from-literal=MYSQL_ROOT_PASSWORD=rootpassword
```

Now lets generate a yaml file to create a mysql pod

```shell
   kubectl run mysql-pod –image=mysql –dry-run=client -o yaml > mysql-pod.yml
```

Now we have to bind the configmap and secret to mysql pod

```shell
   vim mysql-pod.yml
```

```shell

```


```shell
   kubectl apply -f mysql-pod.yml
```

Now lets expose the pod using clusterIP service, so that we can connect it to phpMyAdmin

```shell
   kubectl expose pod mysql-pod –port=3306 –target-port=3306 –name=mysql-svc
```

### What is phpMyAdmin?

phpMyAdmin is a free software tool written in [PHP](https://www.php.net/), intended to handle the administration of MySQL over the Web. phpMyAdmin supports a wide range of operations on [MySQL](https://www.mysql.com/) and MariaDB. Frequently used operations (managing databases, tables, columns, relations, indexes, users, permissions, etc) can be performed via the user interface, while you still have the ability to directly execute any SQL statement.

**PMA_HOST** define address/host name of the MySQL server

**PMA_PORT** define port of the MySQL server

**PMA_USER and PMA_PASSWORD** define username and password to use only with the config authentication method

Now lets create a phpMyAdmin pod, so that we can manage the MySQL Database.

```shell
   kubectl get svc
   kubectl create cm phpadmin-config –from-literal=PMA_HOST=<clusterip_of_mysql> --from-literal=PMA_PORT=<mysql_port>
   kubectl create secret generic phpadmin-secret –from-literal=PMA_USER=root –from-literal=PMA_PASSWORD=rootpassword
```
Now lets generate a yaml file to create a phpMyAdmin pod

```shell
   kubectl run phpadmin-pod  --image=phpmyadmin –dry-run=client -o yaml > phpadmin-pod.yml
```

Now we have to bind the configmap and secret to phpMyAdmin pod

```shell
   vim phpadmin-pod.yml
```

```shell

```

```shell
   kubectl apply -f phpadmin-pod.yml
```

Now lets expose the pod using NodePort service, so that we can access phpMyAdmin for managing MySQL Database

```shell
   kubectl expose pod phpadmin-pod –type=NodePort –port=8099 –target-port=80 –name=phpadmin-svc
   kubectl get all
```

Access the phpMyAdmin UI using master node public ip and node port

Import the sql data

### What is PHP?

PHP is a server-side scripting language designed for web development, but which can also be used as a general-purpose programming language. PHP can be added to straight HTML or it can be used with a variety of templating engines and web frameworks. PHP code is usually processed by an interpreter, which is either implemented as a native module on the web-server or as a common gateway interface (CGI). 
[wikipedia.org/wiki/PHP](https://en.wikipedia.org/wiki/PHP)

In the index.html change the localhost to mysql clusterip and root user, password,mysql database name

Build the docker file

Push the image
 
```shell
   kubectl run php-app –image=iamsaikishore/phptodoapp
   kubectl expose pod php-app –type=NodePort –port=8088 –target-port=80 –name=phpapp-svc
```

To access the application public ip of master and node port

Do some tasks









