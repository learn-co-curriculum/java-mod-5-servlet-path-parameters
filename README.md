# Servlet Path Parameters

## Learning Goals

- Define Path Parameters
- Implement a servlet that gets client data from path parameters

## Code Along

## Introduction

In the previous lesson we saw how to pass information to a servlet by appending
parameter data to the URL as part of the query string.  In this lesson, we see another
approach where the data is a substring within the URL path.

## Path Parameters

Consider typing the following URL into the browser: `http://localhost:8080/fruits/apple?color=red`.

The `HttpServletRequest` interface has several methods that retrieve different parts of the URL:

| HttpServletRequest Method | Result                                       |
|---------------------------|----------------------------------------------|
| getMethod()               | GET                                          |
| getProtocol()             | HTTP/1.1                                     |
| getServerPort()           | 8080                                         |
| getRequestURL()           | http://localhost:8080/fruits/apple?color=red |
| getRequestURI()           | /fruits/apple                                |
| getPathInfo()             | /apple                                       |
| getQueryString()          | color=red                                    |


A **path parameter** is the variable part of a URI.  

For example, given the following URLs:

- `http://localhost:8080/continents/asia`
- `http://localhost:8080/continents/africa`
- `http://localhost:8080/continents/europe`

The variable part of the URI is the substring after the last slash, i.e. the continent name.
We use curly braces to denote a path parameter, thus the URL with a path parameter
for the continent name could be written as:

`http://localhost:8080/continents/{continent_name}`

Note that there could be several path parameters in a URL, for example:

`http://localhost:8080/continents/{continent_name}/countries/{country_name}`.

However, let's assume there is just a single path parameter for the continent name in the URL:
`http://localhost:8080/continents/{continent_name}`.

We will create a servlet that extracts the `continent_name` path parameter, and generates
as a response a JSON-formatted string describing the continent name, area (km^2), and population:

| URL                                             | Servlet response                                                     |
|-------------------------------------------------|----------------------------------------------------------------------|
| http://localhost:8080/continents/asia           | {"name":"asia","area":44614000,"population":4700000000}              |
| http://localhost:8080/continents/africa         | {"name":"africa","area":30365000,"population":1400000000}            |
| http://localhost:8080/continents/europe         | {"name":"europe","area":10000000,"population":750000000}             |
| http://localhost:8080/continents/north_america  | {"name":"north america","area":24230000,"population":600000000}      |
| http://localhost:8080/continents/south_america  | {"name":"south america","area":17814000,"population":430000000}      |
| http://localhost:8080/continents/oceania        | {"name":"australia/oceania","area":8510900,"population":44000000}    |
| http://localhost:8080/continents/antarctica     | {"name":"antarctica","area":14200000,"population":0}                 |

Note that the URL will contain an underscore in place of a space in the continent name.

## Extracting a path parameter

In the previous lesson we used the `getParameter()`  method to get a parameter value from the query string.
Unfortunately, the `HttpServletRequest` interface does not provide a method to extract a path parameter.
However, we can use the `getRequestURI()` or `getPathInfo()` methods, along with some
string processing, to get a path parameter value. One technique is shown below:

```java
// get the {continent_name} path parameter http://localhost:8080/continents/{continent_name}
String continent_name = req.getRequestURI().substring("/continents/".length());
```

For simplicity, we will assume a correctly formatted URL is provided to the servlet.
Otherwise, an exception would be thrown if the substring `/continents/` is not in the URL.


## Servlet Implementation

Let's implement a new servlet named `ContinentServlet` that will take a GET request
of the form `http://localhost:8080/continents/{continent_name}` and return a JSON
string containing the appropriate continent data (name, area, population).

First, we need to define a class `Continent` as shown:

```java
public class Continent {
  private String name;
  private Integer area;
  private Long population;

  public Continent() {}

  public Continent(String name, Integer area, Long population) {
    this.name = name;
    this.area = area;
    this.population = population;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Integer getArea() {
    return area;
  }

  public void setArea(Integer area) {
    this.area = area;
  }

  public Long getPopulation() {
    return population;
  }

  public void setPopulation(Long population) {
    this.population = population;
  }

  @Override
  public String toString() {
    return "{" +
            "\"name\":" + "\"" + name + "\"" + "," +
            "\"area\":" +  area +  "," +
            "\"population\":" + population +
            "}";
  }
}

```

The `toString()` method returns a JSON-formatted string containing to the instance variable values.

We will also create a new servlet class `ContinentServlet`.  

- The servlet uses a hashmap to map the continent name to a `Continent` class instance.  
- The servlet initializes the hashmap by overriding the `init()` method,
  which is called one time when the servlet is instantiated by the servlet container.
- The servlet overrides the `doGet()` method to get a `Continent` based on the name specified in the path parameter.
  - The response includes an error status if the continent is not in the hashtable.
  - The response includes a success status if the continent is in the hashtable.

```java
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.HashMap;

@WebServlet(urlPatterns="/continents/*")
public class ContinentServlet extends HttpServlet {

    //key is continent name, value is land area (Km^2)
    private HashMap<String, Continent> map;

    // init() is automatically called after the Servlet is instantiated
    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        // data from https://en.wikipedia.org/wiki/Continent
        map = new HashMap<>();
        map.put("asia", new Continent("asia", 44614000, 4700000000L ));
        map.put("africa", new Continent("africa", 30365000, 1400000000L));
        map.put("north_america", new Continent("north america", 24230000, 600000000L ));
        map.put("south_america", new Continent("south america", 17814000, 430000000L ));
        map.put("antarctica", new Continent("antarctica", 14200000, 0L));
        map.put("europe", new Continent("europe", 10000000, 750000000L));
        map.put("oceania", new Continent("australia/oceania", 8510900, 44000000L));
    }

    //http://localhost:8080/continents/{continent_name}
    protected void doGet(
            HttpServletRequest req,
            HttpServletResponse resp)
            throws ServletException, IOException {

      // get the {continent_name} path parameter http://localhost:8080/continents/{continent_name}
      String continent_name = req.getRequestURI().substring("/continents/".length());

      // lookup the continent from the hashmap using the continent name as the key
      Continent continent = map.get(continent_name);

      // generate the response
      if (continent == null) {
        // send error status and message as server response
        resp.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);  //status = 500
        resp.setContentType("text/plain;charset=UTF-8");
        resp.getOutputStream().print(String.format("Continent %s not found", continent_name));
      }
      else {
        // send success status and state formatted as JSON as server response
        resp.setStatus(HttpServletResponse.SC_OK);   //status = 200
        resp.setContentType("application/json;charset=UTF-8");
        resp.getOutputStream().print(continent.toString());  
      }
    }
}
```

We can test the new servlet using the various URLs shown in the table below.
Don't forget to stop and restart the `Tomcat9` plugin before testing the URLs.

| URL                                            | Servlet response                                                  |
|------------------------------------------------|-------------------------------------------------------------------|
| http://localhost:8080/continents/asia          | {"name":"asia","area":44614000,"population":4700000000}           |
| http://localhost:8080/continents/africa        | {"name":"africa","area":30365000,"population":1400000000}         |
| http://localhost:8080/continents/europe        | {"name":"europe","area":10000000,"population":750000000}          |
| http://localhost:8080/continents/north_america | {"name":"north america","area":24230000,"population":600000000}   |
| http://localhost:8080/continents/south_america | {"name":"south america","area":17814000,"population":430000000}   |
| http://localhost:8080/continents/oceania       | {"name":"australia/oceania","area":8510900,"population":44000000} |
| http://localhost:8080/continents/antarctica    | {"name":"antarctica","area":14200000,"population":0}              |
| http://localhost:8080/continents/unknown       | Continent unknown not found.                                      |


## Conclusion

An HTTP client can pass information as part of the URL path.  
A path parameter represents the variable path of a path.
We use curly braces to denote a path parameter.  A servlet can use
the `getRequestURI()` or `getPathInfo()` methods, along with
string processing methods, to get a path parameter value.

## Resources

- [What are path parameters](https://www.abstractapi.com/api-glossary/path-parameters)     
- [HttpServletRequest](https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServletRequest.html)  
