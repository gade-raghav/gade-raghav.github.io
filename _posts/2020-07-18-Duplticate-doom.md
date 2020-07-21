---
layout: default
published: true
---

<article class="page">
<div class="entry">

# Is it easy to write a Weather API ?
This is the first api that I've written , and the following is basically me trying to make stuff simple for you so that you can break it and then learn.
The way I define stuff is just to make you comfortable with it . If you find something complicated , [GOOGLE IT](www.google.com) or you can always reach me.
Let's make mistakes and then learn.

***My way is not the ~~right~~ way, but it works.***

### What you will build
-  You will build a simple weather api which gives current  weather 
-  Later we'll add frontend to it because terminal is intimidating for me. (Haha)

### What you need
- 15 mins to read but couple of days to apply what you read.
- java basics 
- patience 

### Let's get started

code's on [github](https://github.com/gade-raghav/Info-Weather) for reference

We are going to use [openweathermap](https://home.openweathermap.org/). Sign in and get your api-id.

Go to [Spring Initializr](https://start.spring.io/)

Project-type: Maven

Language: Java

Dependecies : Spring Web

Generate it. Unzip it. 

*I'd strictly advice you to use a code editor (VSCode works for me)*

**Few things you'll understand the hard way:**
- pom.xml file has the dependencies. If you need something new, search for the maven dependency online and add the required lines. 
- imports are necessary and tricky.

#### There are 2 major components
1. Current Weather
2. Error handling  (trust me this is necessary)

```bash
 cd  demo/src/main/java/com/example/restservice  
```
What are you going to do?

You're going to make a GET request to openweathermap-api which will return some data. You will use the requred data and return it as a json response. Simple?

First set the environment variable **WEB_APPID**(api-key). It's always good to set api-key as an env variable and then use it in code. Code below is self explinatory except for the fact that this is the main method and SpringApplication.run is supposed to be in this method.WeatheController is class name.

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

Get mapping for get request with 1 parameter that is location. Try-catch block makes it easy to find errors. Connect with url ,  read the response using buffer reader(many other ways to do the same thing ) and convert it to JSONObject and return it.
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

I am very bad at handling errors. Logging is ideal but this is what I came up with to notify me when there's something wrong.(Write this in WeatherController.java)

```java
//WeatherController.java
        @RestController
        class MyErrorController implements ErrorController  {
    
                @GetMapping("/error")
                public String handleError() {
        
                        //do something like logging
                        return String.format("Something is really wrong because all t @RestControllerhe known error related issues have been resolved.In case you see this output please raise an issue in github with all the details.\n Thank you \n-RaghavGade");
                }

                @Override
                public String getErrorPath() {
                        return String.format("/error");
                    }
        }
```
Now , You can run this and make curl request in case there are no errors . Navigate to the base directory "demo" . Run the following command:
```bash
./mvnw spring-boot:run
```
curl request with parameter:
```bash
curl "http://localhost:8080/weather/current?location={city name}"
```

### Finally 
So this is how you can build a weather api. The explainations are not very clear , however they are just clear enough to get you going .

***Experiment with the code--> Things will go wrong --> Try fixing them to get what you want.*** == **Learning Curve**
</div>
</article>	
