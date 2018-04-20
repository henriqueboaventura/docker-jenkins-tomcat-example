# Running Jenkins/Tomcat on docker

This is a POC (proof of concept) of a docker network running a Jenkins container which uses a GitHub repository to build a .war file and publish under another container (on the same network) running Tomcat

## Tech Stack

- Docker (latest version)
- Jenkins (latest version running on Java 8)
- Tomcat (version 7 running on a Java 8)
- GitHub (hosting the repository)

## Running the project

1. Clone the repository
2. Run the following command `docker-compose up`

This will create the following structure:

- Jenkins running on http://localhost:8180
- Tomcat running on http://localhost:8280
- Create a basic user on Tomcat to make deployment (user:
  jenkins/password: jenkins)

## Configuring Jenkins

Once everything is up and running (hopefully), it is time to configure Jenkins.

### 1º Step

After run `docker-compose up`, it will show something like this:

```
jenkins_1  | *************************************************************
jenkins_1  | *************************************************************
jenkins_1  | *************************************************************
jenkins_1  |
jenkins_1  | Jenkins initial setup is required. An admin user has been created and a password generated.
jenkins_1  | Please use the following password to proceed to installation:
jenkins_1  |
jenkins_1  | 74b08e697ce94956b9ef949832bxxxx
jenkins_1  |
jenkins_1  | This may also be found at: /var/jenkins_h

ome/secrets/initialAdminPassword
jenkins_1  |
jenkins_1  | *************************************************************
jenkins_1  | *************************************************************
jenkins_1  | *************************************************************
```

Note the hash, you must copy to the clipboard.

### 2º Step

Open your browser and hit http://localhost:8180, it will ask for a password (the one that you copy on the 1º Step). Paste and click **continue**

### 3º Step

Jenkins will ask for which type of customization, choose **Install Suggested Plugins**. It will download and install all the required plugins and it may take some time.

### 4º Step

All the plugins are now installed, time to configure the default user. I recommend to use something basic and simple to remind, like admin/admin, it is a POC, not a production product. Remember, you will need it later on the process ;)
Click **Save and finish**.

### 5º Step

Our Jenkins is up and running, but we need to make some configurations before start using.

Go to **Manage Jenkins** on the left menu and them on **Global Tool Configuration**.

#### JDK

On the JDK section, click on **Add  JDK**, choose a name (anything), uncheck the config *Install automatically* and fill the field **JAVA_HOME** with `/docker-java-home`

#### Maven

On the Maven section, click on **Add Maven**, choose a name (anything), and let *Install automatically* checked.

Hit apply

Go to **Manage Jenkins** on the left menu once again and them on **Manage Plugin**.
Click on the *Available* tab and them, on the search box, type `deploy to container`. It will show the plugin to install. Select it and click *Install without restart*

And we are done with configuring Jenkins.

## Creating the first job

On the Jenkins home screen, click on **New Item** on the left menu. Choose a name to the Job and select **Freestyle Project** and hit *ok*.

On *Source Code Management* choose **Git** and paste the following URL on the *Repository URL*: https://github.com/kidh0/argentum-web.git. This is the project that we want to build.

On *Build*, click on **Add build step** and choose *Invoke top-level Maven targets*. Select the Maven that you already configured and on **Goals** type `clean package`.

On *Post-build actions*, click on **Add post-build actions** and choose *Deploy war/ear to a container*.
On the field **WAR/EAR files** type `target/argentum-web.war` and in **Context path** type `argentum-web`.
On **Add container**, choose *Tomcat 7.x*.
On **Credentials** click on the *add* button and create a user with `jenkins` as username and password. It will be required to the deployment and them select the created user.
On **Tomcat URL** type `http://tomcat:8080`.

Hit **save**

## Running the first build

Now that we made all the configs required, time to build the project for the first time.

Enter the created job and hit **Build now**. The first time will be slower than the following builds, it will install all the required dependencies, don't worry. 

If we have success on the build (blue dot) we can now go to http://localhost:8280/argentum-web and check if our project exists.
