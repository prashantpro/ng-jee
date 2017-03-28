#Sample Multi module project setup showcasing how Angular and JEE can be combined together

The recommended way to build an Angular 2 application is to make use of Angular CLI tool. Similarly when you work with Java EE projects you typically use Maven as the build tool.

Given the frontend/UI project will be Angular 2 application.
Backend project can be Spring or purely Java EE application (Any web app).

The trick is to take the Angular 2 output (**dist** directory) and map it into the web application project for the UI part.

This sample project setup does just that without any fancy third party plugins.
Lets take this Multi module project structure as an example:

**cd ng-jee** (This is the parent POM project)

    .
    ├── pom.xml
    ├── ngdemo
    │   ├── pom.xml    --- maven pom for angular module
    │   ├── dist       --- This will be Angular output
    │   ├── e2e
    │   ├── karma.conf.js
    │   ├── node_modules
    │   ├── package.json
    │   ├── protractor.conf.js
    │   ├── README.md
    │   ├── src
    │   ├── tsconfig.json
    │   └── tslint.json
    └── webdemo
        ├── pom.xml
        └── src

The parent pom.xml needs to list both modules. The **first** module should be the UI (Angular 2) module followed by the Java/Spring module.

The important section is shown below for 
**ng-jee/pom.xml**

    <packaging>pom</packaging>
    <modules>
        <module>ngdemo</module>
        <module>webdemo</module>
    </modules>

Next, take the angular app, which could be created using CLI like this:

ng new ngdemo

Then place a pom.xml in that same directory **ngdemo/pom.xml**
Having the below build plugins: (This will build the Angular CLI project and generate output in its **dist** folder.

     <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <failOnError>false</failOnError>
                    <filesets>
                        <fileset>
                            <directory>.</directory>
                            <includes>
                                <include>dist/**/*.*</include>
                            </includes>
                            <followSymlinks>false</followSymlinks>
                        </fileset>
                    </filesets>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.5.0</version>
                <executions>
                    <execution>
                        <id>angular-cli build</id>
                        <configuration>
                            <workingDirectory>.</workingDirectory>
                            <executable>ng</executable>
                            <arguments>
                                <argument>build</argument>
                                <argument>--prod</argument>
                                <argument>--base-href</argument>
                                <argument>"/ngdemo/"</argument>
                            </arguments>
                        </configuration>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>


The last piece of the puzzle is to reference this **ngdemo/dist** folder so we can copy the output into our WAR file.

So, here's what needs to be done in **webdemo/pom.xml**

    <build> 
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                    <webResources>
                        <resource>
                            <!-- this is relative to the pom.xml directory -->
                            <directory>../ngdemo/dist/</directory>
                        </resource>
                    </webResources>
                </configuration>
            </plugin>
        </plugins>
    </build>


Build the project from the parent directory **ng-jee**

`mvn clean install`

Then you will see that first it builds the Angular project then Web Project, while doing the build for the latter it also copies the Angular dist contents into the Web projects root. 

So you get something like the below in the WAR/Web project target directory:

/ng-jee/webdemo/target/webdemo-1.0-SNAPSHOT.war


    .
    ├── favicon.ico
    ├── index.html
    ├── inline.d72284a6a83444350a39.bundle.js
    ├── main.e088c8ce83e51568eb21.bundle.js
    ├── META-INF
    ├── polyfills.f52c146b4f7d1751829e.bundle.js
    ├── styles.d41d8cd98f00b204e980.bundle.css
    ├── vendor.fb5e0f38fc8dca20e0ec.bundle.js
    └── WEB-INF
        └── classes



