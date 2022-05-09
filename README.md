---
author: Kalgi Shah
categories:
  - Mobile App Development
draft: false
date: '2022-05-10T00:00:00+03:00'
description: 'In this article, we discuss how to set up the most renowned code analysis tool, SonarQube, on a cloud server and integrate it with Codemagic to automate code quality reporting.'
facebook_image: https://blog.codemagic.io/uploads/covers/codemagic-blog-header-mix-redux-vs-recoil.png
featured: true
popular: false
tags:
  - SonarQube
  - Codemagic
  - Code quality
thumbnail: https://blog.codemagic.io/uploads/covers/codemagic-blog-header-mix-redux-vs-recoil.png
thumbnail_alt: 'How to set up SonarQube and integrate it with Codemagic'
thumbnail_title: 'How to set up SonarQube and integrate it with Codemagic'
title: 'How to set up SonarQube and integrate it with Codemagic'
twitter_image: https://blog.codemagic.io/uploads/covers/codemagic-blog-header-mix-redux-vs-recoil.png
slug: sonarqube-integration
related_articles:
  - /build-video-streaming-with-flutter-and-mux.md
  - /react-native-apps-testing-end-to-end.md
  - /react-native-getting-started-guide-with-codemagic.md
---

## What is SonarQube?

[SonarQube](https://www.sonarqube.org/) by [SonarSource](https://www.sonarsource.com/) is the leading tool for continuously inspecting the **code quality** and **security of your codebase** and guiding development teams during code reviews. It is an **open-source tool** that has support for [29 programming languages](https://www.sonarqube.org/features/multi-languages/) as of the time of writing this article, and the number is growing.

SonarQube’s key features include:

- **Code quality checkups**: SonarQube checks the overall health of your code and, more importantly, highlights code-related issues. This makes it a great tool for checking code quality.
- **Intelligent bug detection**: SonarQube provides code analyzers and uses powerful path-sensitive dataflow engines that can point out mistakes like null deferences, logical errors, and resource leaks. 
- **Multilanguage support**: SonarQube has more than 29 code analyzers for different languages/platforms, like C/C++, JavaScript, C#, Java, COBOL, PL/SQL, PHP, ABAP, VB.NET, Python, RPG, Flex, Objective-C, Swift, web, and more.
- **DevOps integration**: It can be easily integrated with CI/CD tools using webhooks and REST APIs.

SonarQube can be run on your local machine or as a Docker container. You can also host it on an on-premises or cloud-based server. SonarQube comes in Community, Developer, and Enterprise editions. The **Community Edition** is free and open source. On the other hand, though it is not free, the **Developer Edition** comes with C, C++, Objective-C, Swift, ABAP, T-SQL, and PL/SQL support, [branch analysis](https://docs.sonarqube.org/latest/branches/overview/), and [pull request decoration](https://docs.sonarqube.org/latest/analysis/ci-integration-overview/).

In this article, we will walk you through hosting SonarQube locally and on an AWS EC2 instance, as well as implementing it into a CI/CD pipeline in Codemagic.

{{<general-banner-warm>}}

## Using SonarQube on your local system

You can either download the SonarQube Community/Developer Edition ZIP file from [here](https://www.sonarqube.org/downloads/) or use their [Docker image](https://hub.docker.com/_/sonarqube/). In this article, we will be using **SonarQube Developer Edition**.

You can check the instructions [here](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/) to learn how to install the local instance using the ZIP file.

To use their Docker image for testing, start the Docker server using:

```sh
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```

Now, log in to `http://localhost:9000` with system administrator credentials (login=admin, password=admin).

![](https://blog.codemagic.io/uploads/2022/04/Local_1.png)

Click the **Create Project** button. When asked how you want to create your project, select **Manually**.

![](https://blog.codemagic.io/uploads/2022/04/Local_2.png)

Enter a **Project display name** and a **Project key** and click **Set Up**.

![](https://blog.codemagic.io/uploads/2022/04/Local_3.png)

Under **Provide a token**, enter a token name and click **Generate**. Copy the token and click **Continue**. You will need this while running the analysis CLI command.

![](https://blog.codemagic.io/uploads/2022/04/Local_4.png)

Select your **project’s main language** and follow the instructions.

![](https://blog.codemagic.io/uploads/2022/04/Local_5.png)

SonarQube has a dedicated Gradle plugin called [SonarScanner for Gradle](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-gradle/), which you can use to generate the SonarQube analysis for your Android project.

SonarQube provides **Swift** support in the Developer Edition. Unfortunately, Swift is not supported in the Community edition. However, you can use their [SonarScanner](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/) as a CLI tool for generating the SonarQube analysis for your iOS project. 

Also, you can always [request a free trial of Developer Edition](https://www.sonarqube.org/developer-edition/#language_list) and try it out for yourself.

#### SonarQube Android integration

Integrating SonarQube with an Android project is pretty straightforward. Follow the steps below:

1. Navigate to your `app/build.gradle` and open it.

2. Add the SonarQube Gradle plugin:

```
plugins {
    id "org.sonarqube" version "3.0"
}
```

3. Run project sync if you're using Android Studio or just run `./gradlew --refresh-dependencies` in the terminal.

4. Then run the following command from the terminal:

```sh
./gradlew sonarqube \
  -Dsonar.projectKey=<project_key> \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<login_token>
```

![](https://blog.codemagic.io/uploads/2022/04/Local_6.png)

#### iOS integration

Download SonarScanner from [here](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/), and add the bin directory to the PATH environment variable. In order to achieve that, start **Terminal** and run the following command: 

```bash
cd ~/
vi .bash_profile
```

These commands will open your `bash_profile` in `vi` editor. Then add the following lines at the end:

```
export PATH=$PATH:/Applications/SonarScanner/bin
export PATH=$PATH:/Applications/SonarQube/bin
```

Press `ESC` key and a colon will appear at the bottom-left corner in vi editor.
Enter `wq` to save & quit.

Use the following command to upload the analysis results:

```
sonar-scanner \
  -Dsonar.projectKey=<project_key> \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<login_token>
```

![](https://blog.codemagic.io/uploads/2022/04/Local_7.png)

You will see the code analysis status displayed on the SonarQube dashboard. In order to connect [Codemagic](https://codemagic.io/start/) to your localhost SonarQube, you will need to make it accessible to the internet. In this case, you can use [ngrok](https://ngrok.com/). Download the tool and follow the instructions on their website.

![](https://blog.codemagic.io/uploads/2022/04/Local_9.png)

{{<exitpopup title="Get your free ebook: CI/CD for mobile apps" image="https://blog.codemagic.io/uploads/2020/11/codemagic-ebook-cicd-for-mobile-apps-blog.png" link="https://codemagic.io/ci-cd-ebook/">}}Don't waste valuable developer resources on things that could be automated.{{</exitpopup>}}

## Connecting to SonarQube with Codemagic using an AWS Linux EC2 instance

When you need SonarQube to be available to the whole team and plan to integrate it into the CI/CD pipeline, hosting it on the server is the best option. Here, we will look into how to deploy SonarQube on an AWS EC2 instance and integrate it with Codemagic to generate a code analysis of your Android and iOS projects.

We need to get a **Linux EC2 server** up and running with **sudo privileges** before installing a sonar server. You can use a machine type of **t2 medium** or larger, as we need **at least 3 GB of RAM** to run SonarQube efficiently. Also, add a custom **TCP security rule** for the EC2 instance to allow **inbound traffic** to the selected SonarQube port (default: `9000`).

#### Step 1: Set up the AWS Linux EC2 instance

Connect to the EC2 instance using a secure shell:

```sh
ssh -i <<path to your .pem file>> ec2-user@<<ip address of your EC2>>
```

Update the system packages on Amazon Linux 2:

```sh
sudo yum update
```

Install `vim`, `wget`, and `curl` on Amazon Linux 2:

```sh
sudo yum install vim wget curl -y
```

Increase the `vm.max_map_count` kernel, `file descriptor`, and `ulimit` for the current session at runtime:

```sh
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

If you want to increase these permanently, open the `limits.conf` config file and insert the proper values, as shown below:

```
 sudo nano /etc/security/limits.conf
 sonarqube   -   nofile   65536
 sonarqube   -   nproc    4096
```

Install Java. We need JDK 11 or higher to run SonarQube 7.9, which we are using in this blog post.

```sh
sudo amazon-linux-extras install java-openjdk11
java -version  // To check java version
```

You can also install **OpenJDK 11** using `curl`.

#### Step 2: Install and configure PostgreSQL 13 on Amazon Linux 2 for SonarQube

We have to set up a database for SonarQube to save the report analysis. This also helps in maintaining the report versions. We will be using PostgreSQL as our database, which can be configured on EC2: 

First, enable the EPEL repository on Amazon Linux 2 using the command below:

```sh
sudo amazon-linux-extras install epel
```

Add the PostgreSQL 13 repo in Amazon Linux 2:

```sh
sudo tee /etc/yum.repos.d/pgdg.repo<<EOF
[pgdg13]
name=PostgreSQL 13 for RHEL/CentOS 7 - x86_64
baseurl=https://download.postgresql.org/pub/repos/yum/13/redhat/rhel-7-x86_64
enabled=1
gpgcheck=0
EOF
```

Now let’s install and initialize PostgreSQL 13 on Amazon Linux using the command below:

```sh
sudo yum install postgresql13 postgresql13-server
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable --now postgresql-13
```

```
sudo systemctl status postgresql-13  //To check PostgreSQL service
```

Change the password for the default PostgreSQL user:

```sh
sudo passwd postgres
su - postgres  //Switch to postgres user
```

Create a new user by typing:

```sh
createuser sonar
```

Switch to the PostgreSQL shell:

```sh
psql
```

Create a user and database for sonar:

```
ALTER USER sonar WITH ENCRYPTED password 'sonar_password';
CREATE DATABASE sonarqube OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
```

Exit the PostgreSQL shell:

```
\q
```

Switch back to the sudo user by running the exit command:

```
exit
```

#### Step 3: Install SonarQube on Amazon Linux 2

Now that we have all the prerequisites in place, we are going to download the binaries for SonarQube and use them to install it.

> Note: Please be careful when selecting the edition that you are going to install. As mentioned before, we are going to use the Developer Edition in this article. More details can be found [here](https://www.sonarqube.org/downloads/). If you want to install a different edition, right-click on the respective download button, and copy the link address.

```sh
sudo wget https://binaries.sonarsource.com/CommercialDistribution/sonarqube-developer/sonarqube-developer-9.3.0.51899.zip
```

Unzip the SonarQube setup file and move it to the `/opt` directory:

```sh
sudo unzip sonarqube-developer-9.3.0.51899.zip
sudo mv -v sonarqube-9.3.0.51899 /opt/sonarqube
```

#### Step 4: Configure SonarQube on Amazon Linux 2

Running the SonarQube instance as a root user causes it to stop running. You can create a new group and user to overcome this issue. 

Create the group first. We’ll call it `sonar`:

```sh
sudo groupadd sonar
```

Now, add the user with directory access:

```sh
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar 
sudo chown -R sonar:sonar /opt/sonarqube
```

Open the SonarQube configuration file using your favorite text editor. We’ll be using `nano` in this example:

```sh
sudo nano /opt/sonarqube/conf/sonar.properties
```

Find the following lines:

```
#sonar.jdbc.username=
#sonar.jdbc.password=
```

Uncomment and type the PostgreSQL database username and password that we created in the steps above. Add the PostgreSQL connection string.

```
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

In the sonar script file, uncomment `RUN_AS_USER` and change it to `RUN_AS_USER=sonar`.

```
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
```

Type `CTRL+X` to save and close the file.

#### Step 5: Start SonarQube

Now, it’s time to start SonarQube.

Switch to the `sonar` user:

```sh
sudo su sonar
```

Move to the script directory:

```sh
cd /opt/sonarqube/bin/linux-x86-64/
```

Run the script to start SonarQube:

```sh
./sonar.sh start 
```

Check SonarQube’s running status:

```sh
./sonar.sh status
```

To check the **SonarQube logs**, navigate to the `/opt/sonarqube/logs/sonar.log` directory.

#### Step 6: Configure the `systemd` service for SonarQube

First, stop the SonarQube service, as we started it manually using the steps above. Navigate to SonarQube’s installation path:

```sh
cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh stop
```

Create a `systemd` service file for SonarQube to run at system startup:

```sh
sudo nano /etc/systemd/system/sonar.service
```

Add the following lines:

```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

Save and close the file.

Now, let’s stop the SonarQube script we started to run:

```sh
./sonar.sh stop
```

Start the SonarQube daemon by running:

```sh
sudo systemctl start sonar
```

Enable the SonarQube service to automatically start at system startup:

```sh
sudo systemctl enable sonar
sudo systemctl status sonar
```

#### Step 7: Access the SonarQube UI

If you are already inside your instance, you can get the public IP of your Linux EC2 instance using the command below:

```
curl -s v4.ident.me
```

Your SonarQube should be up now. You can access the SonarQube UI at `http://<<EC2 instance public ip>>:9000/sonarqube`. By default, the credentials remain `login=admin` and `password=admin`.

![](https://blog.codemagic.io/uploads/2022/04/aws_1.png)

The next step is to add SonarQube to our CI/CD environment so that code quality checks can be automatically triggered by certain events.

{{<newsletter-banner-email-only>}}

#### Step 8: Using SonarQube with Codemagic

We can easily integrate [SonarQube with Codemagic](https://docs.sonarqube.org/latest/analysis/codemagic/) using the [codemagic.yaml](https://docs.codemagic.io/yaml/yaml-getting-started/) file. Codemagic recently worked with Christophe Havard (Product Manager at SonarSource) to add Codemagic to the list of supported CIs for branch and pull-request detection. You can check the SonarQube release notes [here](https://jira.sonarsource.com/browse/SONAR-15412). 

To integrate SonarQube with Codemagic, we will need to set the `SONAR_TOKEN`, `SONARQUBE_URL`, and `SONAR_PROJECT_KEY` environment variables in the Codemagic UI, as shown below. Mark the environment variables as **secure**, and add the respective **group** (`sonarqube`) to the codemagic.yaml file.

![](https://blog.codemagic.io/uploads/2022/04/aws_2.png)

Also, navigate to your `app/build.gradle` and add the SonarQube Gradle plugin:

```
plugins {
    id "org.sonarqube" version "3.0"
}
```

Let’s define the build pipeline script in the codemagic.yaml file for both the Android and iOS projects. In these examples we're using a premium Codemagic's Mac Pro build machine. We want our code quality checks to be triggered on both commits and pull requests, so we specify that in the [triggering](https://docs.codemagic.io/yaml/yaml-getting-started/#triggering) section. You can also reference the sample [Android](https://docs.codemagic.io/yaml-quick-start/building-a-native-android-app/#android-workflow-example) and [iOS](https://docs.codemagic.io/yaml-quick-start/building-a-native-ios-app/#ios-workflow-example) YAML file configurations.

#### Android project with SonarQube integration

```yaml
workflows:
  android-workflow:
    name: Android Workflow
    instance_type: mac_pro
    cache:
      cache_paths:
        - ~/.sonar
    environment:
      groups:
        - sonarqube # includes SONAR_TOKEN, SONARQUBE_URL, SONAR_PROJECT_KEY
    triggering:
      events:
        - push
        - pull_request
      branch_patterns:
        - pattern: '*'
          include: true
          source: true
    scripts:     
      - name: Build Android app
        script: |
          ./gradlew assembleDebug
      - name: Generate and upload code analysis report
        script: |
           ./gradlew sonarqube 
           -Dsonar.projectKey=$SONAR_PROJECT_KEY 
           -Dsonar.host.url=$SONARQUBE_URL 
           -Dsonar.login=$SONAR_TOKEN
```

Once the build is successful, you can check your code analysis on the SonarQube UI. 

![](https://blog.codemagic.io/uploads/2022/04/aws_3.png)

#### iOS project with SonarQube integration

For the iOS build analysis, we first have to download and add the [SonarScanner](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/) to the path. 

SonarScanners running in Codemagic can automatically detect branches and merge or pull requests in certain jobs.

```yaml
workflows:
  ios-workflow:
    name: ios_workflow
    instance_type: mac_pro
    cache:
      cache_paths:
        - ~/.sonar
    environment:
      groups:
        - sonar
      vars:
        XCODE_WORKSPACE: "Sonar.xcodeproj"  # PUT YOUR WORKSPACE NAME HERE
        XCODE_SCHEME: "Sonar" # PUT THE NAME OF YOUR SCHEME HERE
      xcode: latest
      cocoapods: default
    triggering:
      events:
        - push
        - pull_request
      branch_patterns:
        - pattern: '*'
          include: true
          source: true
    scripts:
      - name: Run tests
        script: |
          xcodebuild \
          -project "$XCODE_WORKSPACE" \
          -scheme "$XCODE_SCHEME" \
          -sdk iphonesimulator \
          -destination 'platform=iOS Simulator,name=iPhone 12,OS=15.4' \
          clean build test CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
      - name: Build debug app
        script: |
          xcodebuild build -project "$XCODE_WORKSPACE" \
          -scheme "$XCODE_SCHEME" \
          CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
      - name: Sonar
        script: |
            # download and install the SonarScanner
            wget -O $FCI_BUILD_DIR/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.4.0.2170-macosx.zip
            unzip $FCI_BUILD_DIR/sonar-scanner.zip
            mv sonar-scanner-* sonar-scanner
      - name: Coverage tests
        script: |
            xcodebuild \
            -project "$XCODE_WORKSPACE" \
            -scheme "$XCODE_SCHEME" \
            -sdk iphonesimulator \
            -destination 'platform=iOS Simulator,name=iPhone 11 Pro,OS=15.4' \
            -derivedDataPath Build/ \
            -enableCodeCoverage YES \
            clean build test CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
      - name: convert coverage report to sonarqube format
        script: |
            bash xccov-to-sonarqube-generic.sh Build/Logs/Test/*.xcresult/ > sonarqube-generic-coverage.xml
      - name: Generate and upload code analysis report
        script: |
            export PATH=$PATH:$FCI_BUILD_DIR/sonar-scanner/bin
            sonar-scanner \
            -Dsonar.projectKey=$SONAR_PROJECT_KEY \
            -Dsonar.host.url=$SONARQUBE_URL \
            -Dsonar.c.file.suffixes=- \
            -Dsonar.cpp.file.suffixes=- \
            -Dsonar.coverageReportPaths=sonarqube-generic-coverage.xml \
    artifacts:
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.app
      - /tmp/xcodebuild_logs/*.log
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.dSYM
    publishing:
      email:
        recipients:
            - kalgi@nevercode.io
```

**Coverage reports** cannot be uploaded in this format, so you’ll have to use the following script to convert it from an `.xcresult` to an `.xml` file:

**File name: xccov-to-sonarqube-generic.sh**
```sh
#!/usr/bin/env bash
set -euo pipefail

function convert_file {
  local xccovarchive_file="$1"
  local file_name="$2"
  echo "  <file path=\"$file_name\">"
  local line_coverage_cmd="xcrun xccov view"
  if [[ $@ == *".xcresult"* ]]; then
    line_coverage_cmd="$line_coverage_cmd --archive"
  fi
  line_coverage_cmd="$line_coverage_cmd --file \"$file_name\" \"$xccovarchive_file\""
  eval $line_coverage_cmd | \
    sed -n '
    s/^ *\([0-9][0-9]*\): 0.*$/    <lineToCover lineNumber="\1" covered="false"\/>/p;
    s/^ *\([0-9][0-9]*\): [1-9].*$/    <lineToCover lineNumber="\1" covered="true"\/>/p
    '
  echo '  </file>'
}

function xccov_to_generic {
  echo '<coverage version="1">'
  for xccovarchive_file in "$@"; do
    local file_list_cmd="xcrun xccov view"
    if [[ $@ == *".xcresult"* ]]; then
      file_list_cmd="$file_list_cmd --archive"
    fi
    file_list_cmd="$file_list_cmd --file-list \"$xccovarchive_file\""
    eval $file_list_cmd | while read -r file_name; do
      convert_file "$xccovarchive_file" "$file_name"
    done
  done
  echo '</coverage>'
}

xccov_to_generic "$@"
```

Run this script using:

```sh
bash xccov-to-sonarqube-generic.sh Build/Logs/Test/*.xcresult/ > sonarqube-generic-coverage.xml
```

Pass the result to SonarQube by specifying the following properties:

```
-Dsonar.cfamily.build-wrapper-output.bypass=true \
-Dsonar.coverageReportPaths=sonarqube-generic-coverage.xml \
-Dsonar.c.file.suffixes=- \
-Dsonar.cpp.file.suffixes=- \
-Dsonar.objc.file.suffixes=-
```

![](https://blog.codemagic.io/uploads/2022/04/aws_4.png)

And that’s it! We have successfully integrated SonarQube with Codemagic to run for both Android and iOS projects. Before we wrap up, let’s discuss two minor details you may have noticed in the YAML configurations above that are related to automatic triggering and caching.

### Automatically detecting pull requests

For SonarQube to automatically detect pull requests when using Codemagic, you need to add an event in the triggering section of your `codemagic.yaml` file, as shown in the following snippet:

```yaml
    triggering:
      events:
        - pull_request
```

You may have noticed this in the YAML scripts above. But that’s not all: For **triggering** to work, you also need to set up a [webhook](https://docs.codemagic.io/configuration/webhooks/) between Codemagic and your repository (e.g., Bitbucket, GitHub).

### Caching the .sonar folder

Caching the `.sonar` folder will save build time on subsequent analyses. To do this, add the following snippet to your `codemagic.yaml` file:

```yaml
    cache:
      cache_paths:
        - ~/.sonar
```

### Conclusion

Integrating SonarQube with Codemagic is really simple when using the `codemagic.yaml` file. In this post, we have covered the basic configuration needed to generate the code analysis report, but there are various other properties that you can specify using SonarQube, especially with SonarQube Developer Edition. However, even the free tier may be enough to significantly improve the code quality of your project by automating reporting with SonarQube and Codemagic.

Try it! And if you have any questions or suggestions, we’re always happy to hear them on [our Slack](slack.codemagic.io) or on Twitter (just tag [@codemagicio](https://twitter.com/codemagicio)).

{{< contact-form title="Contact us to have a personal demo of the SonarQube integration with Codemagic!" subtitle="Fill in the form, and our customer engineering team will get back to you" >}}

#### Learn more

- [SonarQube documentation](https://docs.sonarqube.org/latest/)
- [SonarQube branch analysis](https://docs.sonarqube.org/latest/branches/overview/)
- [SonarQube pull request decoration](https://docs.sonarqube.org/latest/analysis/ci-integration-overview/)
- [GitHub repo for Android project](https://github.com/codemagic-ci-cd/codemagic-sample-projects/tree/main/integrations/sonarqube_integration_demo_project/Android)
- [GitHub repo for iOS project](https://github.com/codemagic-ci-cd/codemagic-sample-projects/tree/main/integrations/sonarqube_integration_demo_project/Sonar)
- [Codemagic documentation for Android project](https://docs.codemagic.io/yaml-quick-start/building-a-native-android-app/)
- [Codemagic documentation for iOS project](https://docs.codemagic.io/yaml-quick-start/building-a-native-ios-app/)
