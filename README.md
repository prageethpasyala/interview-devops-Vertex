# Correvate's Devops engineer test

Support files for Correvate's Devops engineer test

# 1 - Code test
----
## 1.0 - Familiarizing with the project

1. Open your terminal and navigate to the directory where your project is located (my-api/).
   
2. Execute the following command to generate the JAR package while skipping the unit tests:
   ```mvn clean package -DskipTests``` OR ```mvn clean package``` <br />
   _This command cleans the project, compiles the source code, and packages it into a JAR file_

3. After generating the JAR package, you can run the .jar file using the following command:
   ```java -jar target/<my-api jar file name>.jar``` <br />
    _find the correct .jar file name in the 'target/' directory_

4. For each of the following commands executed, you can capture the full output logs by redirecting the output to a log file:
   ### Capture output log for JAR generation
    ```sh 
    mvn clean package -DskipTests > jar_generation_log.txt
    ```

   ### Capture output log for running the JAR
    ```sh 
    java -jar target/my-api.jar > jar_execution_log.txt
    ```
   _This will create two log files (jar_generation_log.txt and jar_execution_log.txt) in my-api folder containing the full output logs for each command_

5. You can test the application status in local browser
   ```http://localhost:8080/actuator/health```
   result should be <br />
   ```json
   {
    "status": "UP"
   }
   ```

## 1.1 CI Pipeline
Below is a sample Jenkinsfile that defines a pipeline script using the Jenkins declarative pipeline syntax. <br />

```
-Jenkinsfile-

pipeline {
    agent any

    stages {
        stage('Update Version') {
            steps {
                script {
                    def TIMESTAMP = sh(script: 'date +%s', returnStdout: true).trim()
                    def newVersion = "1.0.1.${TIMESTAMP}"
                    
                    // Update project version
                    sh "mvn versions:set -DnewVersion=${newVersion}"
                }
            }
        }

        stage('Build and Test') {
            steps {
                // Use Maven to build and test
                sh "mvn clean install"
            }
        }

        stage('Execute JAR') {
            steps {
                // Assuming your JAR file is located at target/ directory
                // Change 'your-jar-file-name.jar' to your actual JAR file name
                sh "java -jar target/your-jar-file-name.jar"
            }
        }
    }
}

post {
    always {
        // Archive build artifacts or perform cleanup if needed
        archiveArtifacts artifacts: "**/target/*.jar", allowEmptyArchive: true
    }
}

```



## 1.2 - Docker
1. Update pom.xml for Docker Plugin:
    Add the docker-maven-plugin configuration in the pom.xml file to build a Docker image and include the JAR file.
2. Create a Dockerfile in the /my-api parent directory to define the Docker image configuration. (Cannot define the Dockerfile inside src/main/docker ; docker cannot copy files outside of the its parent folder, so Dockerfile should be in the parent directory)
   
   ```docker
      FROM eclipse-temurin:18-jdk-alpine
      VOLUME /tmp
      COPY target/*.jar app.jar
      ENTRYPOINT ["java","-jar","/app.jar"]
    ```
    
3. Execute the following command to generate the JAR package while skipping the unit tests:
   ```mvn clean package -DskipTests``` OR ```mvn clean package``` <br />


   
#### make sure your docker demon is running before the execute following commands ####

4. Then we can build an image with the following command:
   ```sh
   docker build -t myorg/myapp .
   ```

5. Then we can run it by running the following command:
   ```
   docker run -p 8080:8080 myorg/myapp
   ```
6. Update the README.md file to include clear instructions for building and running the project and run the command:


## 1.3 - Docker compose & logging
To achieve the objectives described in step 1.2 of setting up the Dockerized application and integrating it with the logging stack <br />
1. Navigate to the logging-stack/ directory in your terminal and execute the following command to start the logging stack:
   ```bash
   docker-compose up -d
   ```
   _This will start the Elasticsearch, Kibana, Fluentbit, and your new service containers._

2. To capture the full output logs of the Docker Compose command, run:
   ```bash 
   docker-compose logs > ELK.log
   ```
   _This will save the logs in the ELK.log file_

3. Access Kibana at ```http://localhost:5601``` in your web browser

<br />
<br />

----
# 2 - DevOps Process


Here are the steps involved in defining a Software Development Life Cycle (SDLC) for a SaaS (Software as a Service) application
```
1. Gather and analyze requirements from stakeholders, define project scope, and plan the development process.
   Tools: Jira, Trello, Asana for project management, and Confluence for documentation.


2. Create the application architecture, database schema, and design the user interfaces.
   Tools: Diagramming tools like Lucidchart, draw.io. UI/UX design tools like Figma or Adobe XD.


3. Choose the programming languages, frameworks, libraries, and tools to be used. Set up development, staging, and production environments.
   Tools: Version control (Git), Infrastructure as Code (Terraform, CloudFormation), and CI/CD tools (Jenkins, GitLab CI/CD).


4. Write code according to the design specifications. Develop the front-end, back-end, and any required integrations.
   Tools: Integrated Development Environments (IDEs) like Visual Studio Code, IntelliJ IDEA, and collaboration tools like Slack, Microsoft Teams.


5. Perform unit testing, integration testing, and end-to-end testing using automated testing frameworks.
   Tools: Testing frameworks like JUnit, Selenium, pytest, and code coverage tools like JaCoCo.


6. Review code changes to ensure quality, adherence to coding standards, and best practices.
   Tools: Code review tools like GitHub Pull Requests, GitLab Merge Requests, Crucible.


7. Set up automated build and deployment pipelines to ensure frequent integration and deployment of code changes.
   Tools: CI/CD tools like Jenkins, Travis CI, GitLab CI/CD, and containerization tools like Docker and Kubernetes.


8. Configure monitoring, logging, and error tracking systems to monitor application health and performance.
   Tools: Monitoring tools like Prometheus, Grafana, logging solutions like ELK stack (Elasticsearch, Logstash, Kibana), and APM tools like New Relic.


9. Perform security assessments, penetration testing, and ensure compliance with security standards.
   Tools: Security testing tools like OWASP Zap, SonarQube for code analysis, and compliance tools like Nessus.


10. Engage stakeholders to perform UAT, ensuring the application meets user expectations.
   Tools: Collaboration tools like Microsoft Teams, Slack, or Zoom for communication during UAT.


11. Deploy the application to production. Monitor for issues and provide ongoing support and updates.
   Tools: CI/CD tools for deployment, incident management tools like PagerDuty, and version control for managing releases.

```

<br />
<br />

-----
# 3 - AWS & Solution drafting

here's a first draft of the architecture and a high-level checklist for migrating the current setup to use the new AWS Elastic Search (ES) domain
```
**Draft architecture:**

1. AWS Elastic Search Domain:
   Set up a new AWS Elastic Search domain in the main AWS account. This will serve as the central log aggregation point.

2. Log Forwarding:
   Configure each environment (qa and prod) to forward logs to the AWS Elastic Search domain. Use CloudWatch Logs subscription filters to forward logs from ECS Fargate containers and EC2 instances.

3. Log Ingestion:
   Use Logstash or AWS Lambda to process and format the logs before they are ingested into the Elastic Search domain. This ensures consistent and meaningful log data.

4. Kibana Dashboard:
   Set up Kibana, part of the Elastic Stack, to visualize and analyze the logs. Create custom dashboards for better visibility and troubleshooting.
```

```

**High-Level Checklist:**

1. Create Elastic Search Domain:
   Set up an AWS Elastic Search domain in the main AWS account.
   Configure access policies and IAM roles for log forwarding.

2. Update Logging Configuration:
   Modify the logging configurations of RabbitMQ, Java REST API, microservices, and EC2 ASG to forward logs to CloudWatch Logs.

3. Configure CloudWatch Logs Subscription Filters:
   Set up subscription filters for each log group to send logs to a Lambda function or an SQS queue.
   

4. Set Up Elastic Search Indexing:
   Configure the index templates and mappings for Elastic Search to properly index the incoming log data.

5. Configure Elastic Search Access:
   Ensure that the Elastic Search domain has the necessary security groups, access policies, and IAM roles for log ingestion.

6. Create Kibana Dashboard:
   Set up Kibana and configure it to connect to the Elastic Search domain.
   Create custom dashboards and visualizations to monitor and analyze the logs effectively.

7. Testing and Validation:
   Deploy the updated setup in a testing environment to ensure that logs are properly forwarded, processed, and indexed.

8. Rollout to Production:
   Once validated, roll out the updated setup to the production environment.

9. Documentation and Training:
   Update documentation and provide training to the team on using Kibana, querying logs, and troubleshooting.

10. Monitoring and Maintenance:
   Set up monitoring for the Elastic Search domain, log forwarding, and log processing.
   Regularly monitor and maintain the setup to ensure smooth log aggregation.
```