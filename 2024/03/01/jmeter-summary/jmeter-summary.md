---
title: jmeter summary
date: 2024-03-01 17:27:59
tags:
---
## How to write a jmeter script to generete complex request payload and parse response body
1. To specify the java version to run the script: set java home as java8 or java17 
2. Go to the installation directory. e.g: /usr/local/Cellar/jmeter/5.6.2/libexec/bin in my mac
3. Run the java package to launch the app
    ```
    java -jar ApacheJMeter.jar
    ```
4. Add a JSR223 Preprocessor or JSR223 PostProcessor in the script 
5. JSR223 will use groovy language to write the java scripts 

### How to generate the complex request and parse response payload ?
For example: our scenarios needs to implement SCEP protocol for test, we can add dependency packages in the script, 
and use the library to help us implement the request payload easily.

1. add dependency jar package into JMeter external libs directory:  /usr/local/opt/jmeter/libexec/lib/ext
2. restart Jmeter app to load the external jar packages
3. write your own processor groovy handling functions by importing the jar packages 
```groovy
import org.bouncycastle.cms.*;
CMSSignedData cmsSignedData = new CMSSignedData(responseDat);
```
4. jmeter provides some built-in functions to help us: 
      * ctx: use ctx to get the response body / headers for handling
      ```
        ctx.getPreviousResult().getResponseHeaders()
        ctx.getPreviousResult().getResponseData();
      ```
      * vars: use vars to set and get the variables and pass between the scripts
      ```groovy
        vars.put("token", "xxxx");
        vars.get("token")
        Object object = new Object();
        vars.putObject("object", object)
        Object newObject = (Object)vars.getObject("object");
      ```
      * log: use log to print the logs in the jmeter console
      ```groovy
        log.info("this is a log message")
      ```
      go to Jmeter app, Options --> Log Viewer to check the logs
   <img alt="jmeter_logview.png" src="/Users/yueh/Hexo/YueBlog/source/_posts/jmeter_logview.png"/>
      * SampleRequest: use SampleRequest to get the request body / headers for handling
      ```groovy
        SampleResult.setResponseHeaders("response headers");
        SampleResult.setResponseData("response data");
      ```

### How to extract common functions into library for jmeter
refer to: https://dzone.com/articles/how-to-reuse-your-jmeter-code-with-jar-files-and-s
1. create a new groovy project in intellij idea 
![img.png](jmeter-summary/create_groovy_lib_project.png)
2. open project structure in intellij app, go to Modules, add the dependency jars used for the groovy classes, here I put the dependency libs in directory "libs"
![img.png](jmeter-summary/project_depedency.png)
3. add a new groovy class in the new project
4. open project structure in intellij app, go to Artifacts, select "From modules with dependencies" to build the artifact
![img.png](jmeter-summary/artifact_configuration.png)
5. Go to intellij – Build – Build artifacts to generate the jar library
6. Go to project directory, a new "out" directory generated with the artifacts
7. Copy the jar into the JMeter ext library and restart JMeter app to load the jar

### How to avoid breaking the test if response status code is not 200
Jmeter would mark the request as failed if the response status code is not 200, but sometimes we are expecting the other response code
1. Add a Response Assertion to the request 
2. Set the response code to the expected code 
3. Set the response assertion to "Ignore Status" to avoid breaking the test
![img.png](jmeter-summary/response_code_not_200.png)

### How to add custom assertion to the response 
In addition to checking the response code, we also need to check the response body to make sure the test is correct 
1. Add a JSR223 PostProcessor for parsing the response body 
2. Extract the value in response body and check whether it is expected 
3. If not, set the request as false
```groovy
prev.setSuccessful(false)
```
4. Add a JSR223 Assertion to the request
```groovy
if( !prev.isSuccessful() ) {
	AssertionResult.setFailure(true);
}

```
5.In thread group settings, set the Action as "Stop Thread" when the assertion result is failed 
![img.png](jmeter-summary/action_after_sample_error.png)

      
      

   

