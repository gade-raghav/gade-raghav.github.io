---
layout: default
published: true
---
This is the first API that I've written, and the following is me trying to make stuff simple for you so that you can break it and then learn.
The way I define stuff is just to make you comfortable with it. If you find something complicated, [GOOGLE IT](www.google.com) or you can always reach me.
Let's make mistakes and then learn.

***My way is not the ~~right~~ way, but it works.***

### What you will build
-  You will build a simple weather API which gives current  weather 
-  Later we'll add frontend to it because the terminal is intimidating for me. (Haha)

### What you need
- 15 mins to read but a couple of days to apply what you read.
- java basics 
- patience 

### Let's get started

code's on [github](https://github.com/gade-raghav/Info-Weather) for reference

We are going to use [openweathermap](https://home.openweathermap.org/). Sign in and get your api-id.

Go to [Spring Initializr](https://start.spring.io/)
- Project-type: Maven
- Language: Java
- Dependencies: Spring Web

Generate it. Unzip it. 

*I'd strictly advise you to use a code editor (VSCode works for me)*

**Few things you'll understand the hard way:**
- pom.xml file has the dependencies. If you need something new, search for the maven dependency online and add the required lines. 
- imports are necessary and tricky.

#### There are 3 major components
1. Current Weather
2. Returning JSON response
3. Error handling 

```bash
 cd  demo/src/main/java/com/example/restservice #asuming that demo is the directory name 
```

First and foremost, you need the following imports (half of the trouble ends here). However, to make some imports you need to add dependencies(in pom.xml file).

This is how my pom.xml looks.
```xml
//pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
    		<artifactId>mavenreactjsspringboot</artifactId>
    		<version>0.0.1-SNAPSHOT</version>
    		<name>mavenreactjsspringboot</name>
	    <description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20150729</version>
</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
	</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
    </plugins>
</build>
</project>  
```

These are the imports in WeatherController.java

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.List;
import java.util.Optional;
import org.json.JSONArray;
import org.json.JSONObject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.error.ErrorController;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
```


What are you going to do?

You'll make a GET request to openweathermap-api which will return some data. You will use the required data from the output of openweathermap-api and return it as a JSON response. Simple?


First set the environment variable **WEB_APPID**(api-key). It's always good to set api-key as an env variable and then use it in code. The code below is self-explanatory except for the fact that this is the main method and SpringApplication.run is supposed to be in this method.WeatheController is a class name.

```java
//WeatherController.java
public static void main(String[] args) {
                  SpringApplication.run(WeatherController.class, args);
		  key=System.getenv("WEB_APPID");
		  if (key == null){
			  System.out.print("\nPlease set an environment variable as instructed in the README.md file\n");
			  System.exit(1);
                  }
	}
```

Get mapping for "get" request with 1 parameter that is location. Try-catch block makes it easy to find errors. Connect with URL,  read the response using buffer reader(many other ways to do the same thing ) and convert it to JSONObject and return it.
```java
//WeatherController.java

@GetMapping("/weather/current")
	@CrossOrigin(origins="http://localhost:3000")
        public Weather current(@RequestParam(value="location" )String location) {
                try {

                        String url = "http://api.openweathermap.org/data/2.5/weather?q="+location+"&appid="+key; 
                        URL obj = new URL(url);

                        HttpURLConnection con = (HttpURLConnection) obj.openConnection();

                        BufferedReader in = new BufferedReader(
                                        new InputStreamReader(con.getInputStream()));
                        String inputLine;
                        StringBuffer response = new StringBuffer();
                        while ((inputLine = in.readLine()) != null) {
                                response.append(inputLine);
                        }
                        

                        in.close();
                        content = response.toString();

                } 
                catch (Exception e) {
                        System.out.print("ERROR : "+e);
                        return new Weather("Not Found","Not Found","Not Found","Not Found",0);
                }
                        JSONObject root = new JSONObject(content);
                        //coordinates
                        //temperature-humidity-pressure
                        JSONObject main = root.getJSONObject("main");
                        //system
                        JSONObject sys = root.getJSONObject("sys");
                        //weather
                        JSONArray wea = root.getJSONArray("weather");
                        JSONObject weas = wea.getJSONObject(0);
                        
                        return new Weather(
                                        weas.getString("main"),
                                        weas.getString("description"),
                                        sys.getString("country"),
                                        root.getString("name"),
                                        main.getInt("temp")
                                        );
                }
```
Openweathermap API returns a lot of data regarding weather which might not be used by you, therefore we'll parse it using JSONObject as we did in the above code.

**pro tip**: [this](http://jsonviewer.stack.hu/) helps a lot to get a perspective of your JSON response and then parsing becomes easy. Initially try to return the whole content from API and then try to parse it.JSONObject code is self-explanatory.

So, where did the weather object pop up from? You need to write some code for that. In my case it's Weather.java and depending upon my response I've written the following lines.
```java
//Weather.java
package com.example.restservice;

public class Weather{
	private String main;
	private String description;
	private float temp;
	private String country;
	private String city;


	public Weather( String main,String description,String country, String city,float temp){
		this.main=main;
		this.description=description;
		this.country=country;
		this.city=city;
		this.temp=temp;
	}
		public String getMain() {
				return main;
		}
		public String getDescription() {
				return description;
		}
		public String getCountry() {
                return country;
        }
        public  String getCity() {
                return city;
        }
		public float getTemp() {
				return temp;
		}
}
```

I am very bad at handling errors. Logging is ideal but this is what I came up with to notify me when there's something wrong.

```java
//WeatherController.java
        @RestController
        class MyErrorController implements ErrorController  {
    
                @GetMapping("/error")
                public String handleError() {
        
                        //do something like logging
                        return String.format("Something is wrong because all t @RestControllerhe known error related issues have been resolved. In case you see this output please raise an issue in GitHub with all the details.\n Thank you \n-RaghavGade");
                }

                @Override
                public String getErrorPath() {
                        return String.format("/error");
                    }
        }
```
Now, You can run this and make a curl request in case there are no errors. Navigate to the base directory "demo". Run the following command:
```bash
./mvnw spring-boot:run
```
curl request with parameter:
```bash
curl "http://localhost:8080/weather/current?location={city name}"
```

### Finally 
So this is how you can build a weather API. The explanations are not very clear, however, they are just clear enough to get you going.

***Experiment with the code--> Things will go wrong --> Try fixing them to get what you want*** == **Learning Curve**
