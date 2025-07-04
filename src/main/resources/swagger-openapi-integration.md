# Swagger (OpenAPI) Documentation Integration for Spring Boot & Jersey (Jakarta RS)

This guide explains how to integrate OpenAPI (formerly Swagger) documentation into a Spring Boot application, covering both standard Spring MVC REST controllers and Jersey (Jakarta RS) REST APIs. It also details how to serve Swagger UI for interactive API exploration.

---

## 1. Spring Boot (Spring MVC) with `springdoc-openapi`

For applications using Spring MVC (the default in Spring Boot), use the [springdoc-openapi](https://springdoc.org/) library, which automatically generates OpenAPI 3 documentation for your REST APIs.

### **Step-by-Step Setup**

#### 1.1 Add Dependency

Add the following to your Maven `pom.xml`:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

#### 1.2 Accessing the Documentation

- **Swagger UI:**  
  [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
- **OpenAPI JSON:**  
  [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)

#### 1.3 Security Configuration

Whitelist the following endpoints in your `WebSecurityConfig` to allow public access (no authentication required):

```java
public static final String SWAGGER_DOC_UI_ENDPOINT = "/swagger-ui/**";
public static final String SWAGGER_DOC_API_ENDPOINT = "/v3/api-docs/**";
public static final String SWAGGER_WEBJARS_ENDPOINT = "/webjars/**";
```

---

## 2. Spring Boot with Jersey (Jakarta RS) REST APIs

If your application uses Jersey (Jakarta RS) to expose REST APIs, the recommended OpenAPI integration options are:

- **SmallRye OpenAPI**
- **swagger-core** with `swagger-jersey2-jaxrs`

This guide covers the `swagger-core` approach.

### **Step-by-Step Setup (swagger-core + Jersey)**

#### 2.1 Add Maven Dependencies

Add the following to your Maven `pom.xml` (ensure you select versions compatible with your Jersey and Spring Boot versions):

```xml
<!-- Swagger Core for Jersey -->
<dependency>
    <groupId>io.swagger.core.v3</groupId>
    <artifactId>swagger-jaxrs2-jakarta</artifactId>
    <version>2.x.x</version>
</dependency>
<dependency>
    <groupId>io.swagger.core.v3</groupId>
    <artifactId>swagger-integration</artifactId>
    <version>2.x.x</version>
</dependency>
```

#### 2.2 Jersey Configuration

Register the Swagger/OpenAPI configuration with your Jersey application to auto-generate the OpenAPI specification (`openapi.json`).  
Example path:  
```
/api/openapi.json
```
Where `/api` is the value of `spring.jersey.application-path` (set in `application.properties`).

#### 2.3 Annotate Resources

Use OpenAPI/Swagger annotations in your Jersey resource classes for richer documentation:
```java
@Operation(summary = "Get user by ID", ...)
@Path("/users/{id}")
public Response getUser(@Parameter(...) @PathParam("id") String id) { ... }
```

#### 2.4 Security Configuration

Whitelist the OpenAPI spec endpoint in your security configuration:
```java
public static final String SWAGGER_JERSEY_ENDPOINT = "/api/openapi.json";
```

---

## 3. Serving Swagger UI with Jersey

Unlike `springdoc-openapi`, Jersey integration does **not** automatically serve Swagger UI.  
You need to manually add the Swagger UI static files.

### **Manual Setup**

#### 3.1 Download Swagger UI

- Download the latest [Swagger UI distribution](https://github.com/swagger-api/swagger-ui/releases).
- Extract the `dist` folder.

#### 3.2 Add to Your Application

Copy the contents of `dist` to your Spring Boot static resources directory:

```
src/main/resources/static/swagger-ui/
```

#### 3.3 Configure Swagger UI

Edit `swagger-ui/index.html` and set the `url` to your OpenAPI JSON endpoint:

```html
<script>
  window.onload = function() {
    window.ui = SwaggerUIBundle({
      url: "/api/openapi.json", // Update if your application path is different
      dom_id: '#swagger-ui',
      presets: [
        SwaggerUIBundle.presets.apis,
        SwaggerUIStandalonePreset
      ],
      layout: "StandaloneLayout"
    });
  };
</script>
```

#### 3.4 Access Swagger UI

Open [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html) in your browser.

#### 3.5 Security Configuration

Whitelist the following endpoints to permit public access:

```java
public static final String SWAGGER_DOC_UI_ENDPOINT = "/swagger-ui/**";
public static final String SWAGGER_JERSEY_ENDPOINT = "/api/openapi.json";
```

---

## 4. Summary Table: Swagger/OpenAPI Endpoints

| Use Case                | OpenAPI JSON URL                        | Swagger UI URL                          | Whitelist Patterns                                          |
|-------------------------|-----------------------------------------|-----------------------------------------|------------------------------------------------------------|
| Spring MVC (springdoc)  | `/v3/api-docs`                          | `/swagger-ui.html`                      | `/swagger-ui/**`, `/v3/api-docs/**`, `/webjars/**`         |
| Jersey (swagger-core)   | `/{spring.jersey.application-path}/openapi.json` | `/swagger-ui/index.html`                | `/swagger-ui/**`, `/api/openapi.json`                      |

---

## 5. Notes

- When using Jersey, **springdoc-openapi** does **not** serve Swagger UI automatically.
- For richer documentation, use OpenAPI annotations such as `@Operation`, `@Parameter`, and `@Schema` in your resource classes.
- Always ensure that your API documentation endpoints are whitelisted in your security configuration for developer access.

---

**References:**
- [springdoc-openapi Documentation](https://springdoc.org/)
- [Swagger Core (swagger-jersey2-jaxrs)](https://github.com/swagger-api/swagger-core)
- [Swagger UI Releases](https://github.com/swagger-api/swagger-ui/releases)