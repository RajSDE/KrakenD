## KrakenD
Krakend is an open-source, high-performance API gateway that helps developers manage, optimize, and secure their APIs. It acts as a middle layer between clients (such as web or mobile applications) and backend services (like microservices, databases, or third-party APIs). Krakend provides several useful features:

1. **Request Aggregation**: It allows you to combine multiple API requests into a single call, reducing the number of round trips between clients and the server.

2. **Rate Limiting**: Krakend can apply rate limits to APIs, ensuring they are protected from abuse by limiting the number of requests within a certain timeframe.

3. **Authentication & Authorization**: It supports integrating JWT tokens and OAuth, enabling you to secure your APIs and manage access control.

4. **Caching**: It supports caching, which helps to reduce the load on backend services by storing responses and serving them from cache when needed.

5. **Transformation**: Krakend allows for request and response transformation, enabling you to modify payloads before sending them to the client or backend.

6. **Load Balancing**: It can distribute traffic across multiple backend services to ensure even load distribution and high availability.

7. **Monitoring**: Krakend integrates with popular monitoring tools like Prometheus, allowing you to track API performance and health.

It's often used in microservices architectures, where multiple services need to be exposed in a unified way to client applications, improving performance and security while simplifying the management of APIs.

## To implement KrakenD for your microservices 
### Follow these steps:

### 1\. **Install KrakenD**

* You can install KrakenD on your local machine or server via different methods such as Docker, binary files, or package managers.

* **Using Docker:**

  ```bash
  bashdocker run -p 8080:8080 devopsfaith/krakend run -c /etc/krakend/krakend.json
  ```

* **Using Binary (for Linux/MacOS):** Download the binary from [KrakenD official website]().

### 2\. **Define the API Gateway Configuration**

KrakenD is configured via a JSON file where you define your endpoints, backends (microservices), transformations, authentication, etc.

Example of a simple configuration (krakend.json):

```json
json{
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/api/v1/users",
      "method": "GET",
      "output_encoding": "json",
      "backend": [
        {
          "url_pattern": "/users",
          "host": ["http://microservice1:8080"],
          "method": "GET",
          "extra_config": {}
        }
      ]
    },
    {
      "endpoint": "/api/v1/orders",
      "method": "GET",
      "output_encoding": "json",
      "backend": [
        {
          "url_pattern": "/orders",
          "host": ["http://microservice2:8080"],
          "method": "GET",
          "extra_config": {}
        }
      ]
    }
  ]
}
```

* Each **endpoint** is a public API that clients will call.
* Each **backend** is the actual microservice behind that endpoint.

### 3\. **Start KrakenD**

Run KrakenD with the configuration file:

```bash
bashkrakend run -c krakend.json
```

### 4\. **Integrate Authentication (JWT)**

You can add JWT authentication to secure your endpoints. Here’s an example with a JWT token validation in the config:

```json
json{
  "extra_config": {
    "github.com/devopsfaith/krakend-jose/validator": {
      "alg": "HS256",
      "jwk-url": "https://your-jwt-server.com/.well-known/jwks.json",
      "cache": true
    }
  }
}
```

### 5\. **Rate Limiting**

Implement rate limiting to protect your services from being overwhelmed by too many requests:

```json
json{
  "extra_config": {
    "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
      "maxRate": 5,
      "capacity": 10
    }
  }
}
```

### 6\. **Request Aggregation**

You can aggregate multiple requests from different microservices into one response:

```json
json{
  "endpoint": "/api/v1/dashboard",
  "backend": [
    {
      "url_pattern": "/users",
      "host": ["http://microservice1:8080"]
    },
    {
      "url_pattern": "/orders",
      "host": ["http://microservice2:8080"]
    }
  ]
}
```

### 7\. **Monitoring and Logging**

Integrate Prometheus or other monitoring tools by enabling monitoring in the configuration:

```json
json{
  "extra_config": {
    "telemetry": {
      "metrics": {
        "collection_time": "30s",
        "prometheus": true
      }
    }
  }
}
```

### 8\. **Deploying KrakenD**

Deploy KrakenD along with your microservices in a containerized environment like Kubernetes or Docker Swarm. You can scale KrakenD horizontally by running multiple instances behind a load balancer.

By using KrakenD, you’ll have a centralized gateway to manage your microservices, providing features like security, load balancing, and performance improvements.

## To define the API Gateway configuration for KrakenD in depth
### Let’s break it down step-by-step, including files, folder structure, and key concepts.

### 1\. **krakend.json – The Configuration File**

The core of KrakenD configuration is the JSON file, typically named `krakend.json`. This file contains all the information KrakenD uses to route requests, secure APIs, manage traffic, and interact with your microservices.

Here's an expanded version of the configuration file structure:

```json
json{
  "version": 3,  // Configuration version
  "name": "My API Gateway",  // The name of your API gateway instance
  "port": 8080,  // The port KrakenD listens on
  "cache_ttl": "300s",  // Cache time to live (TTL)
  "timeout": "3000ms",  // Maximum request timeout
  "endpoints": [  // Define your public-facing API endpoints
    {
      "endpoint": "/api/v1/users",
      "method": "GET",  // HTTP method to be supported (GET, POST, etc.)
      "output_encoding": "json",  // The format of the output (json, xml, etc.)
      "headers_to_pass": ["Authorization"],  // Headers passed to backend
      "extra_config": {},  // Additional configuration (auth, rate limit, etc.)
      "backend": [  // Define the backend services for this endpoint
        {
          "url_pattern": "/users",  // Path on the backend service
          "host": ["http://microservice1:8080"],  // Address of backend microservice
          "method": "GET",  // Method used to fetch data from the backend
          "encoding": "json",  // Format of the backend's response
          "extra_config": {}  // Any additional configuration per backend
        }
      ]
    },
    {
      "endpoint": "/api/v1/orders",
      "method": "GET",
      "output_encoding": "json",
      "backend": [
        {
          "url_pattern": "/orders",
          "host": ["http://microservice2:8080"],
          "method": "GET",
          "encoding": "json"
        }
      ]
    }
  ]
}
```

### Key Elements of the `krakend.json` File:

* **version**: Specifies the KrakenD configuration version, which must be set to 3.

* **name**: The name for this API Gateway instance.

* **port**: The port on which KrakenD will listen for incoming requests (e.g., 8080).

* **cache\_ttl**: How long responses from the backend should be cached in the gateway (optional).

* **timeout**: Maximum time allowed for a backend to respond.

* **endpoints**: This defines the public API endpoints. Each endpoint corresponds to a public route that clients will call.

  Each **endpoint** contains:

  * **endpoint**: The public URL path that the gateway exposes (e.g., `/api/v1/users`).
  * **method**: The HTTP method supported by this route (`GET`, `POST`, etc.).
  * **output\_encoding**: Specifies the response format, usually `json`.
  * **headers\_to\_pass**: Headers that KrakenD will pass from the client to the backend service.
  * **backend**: This is the backend service or microservice that will handle the request. A backend is defined by:
    * **host**: The base URL(s) for the microservice.
    * **url\_pattern**: The path on the microservice to be invoked.
    * **method**: The HTTP method KrakenD will use to call the microservice.
    * **encoding**: Format of the backend’s response (json, xml, etc.).
  * **extra\_config**: Additional configuration such as rate limiting, caching, JWT validation, etc.

### 2\. **Folder Structure**

A typical KrakenD project will have the following folder and file structure:

```bash
bash/krakend-gateway/
├── config/                      # Contains configuration files
│   ├── krakend.json              # Main KrakenD configuration file
│   └── other-config.json         # Optional config for specific modules (e.g., auth)
├── plugins/                     # Optional: Custom plugins for KrakenD
├── logs/                        # Logs directory (optional, based on config)
└── Dockerfile                   # If deploying with Docker
```

* **config/**: Contains all the configuration files for the API Gateway. The `krakend.json` file is the most important file here, containing your API routing logic, security, and traffic control configurations.
* **plugins/**: If you are extending KrakenD with custom Go plugins, you would place them here. KrakenD supports custom-built plugins if its default functionality is not enough for your needs.
* **logs/**: If enabled in your configuration, logs will be stored here. KrakenD provides built-in logging, but it can be extended or redirected to external monitoring tools.

### 3\. **Advanced Configuration Options**

#### Request Aggregation

You can aggregate multiple backend services under a single API endpoint. This allows KrakenD to request data from several microservices, merge the responses, and send a single response back to the client.

Example:

```json
json{
  "endpoint": "/api/v1/aggregated",
  "method": "GET",
  "output_encoding": "json",
  "backend": [
    {
      "url_pattern": "/users",
      "host": ["http://microservice1:8080"],
      "encoding": "json"
    },
    {
      "url_pattern": "/orders",
      "host": ["http://microservice2:8080"],
      "encoding": "json"
    }
  ]
}
```

In this case, `/api/v1/aggregated` will send requests to `/users` on `microservice1` and `/orders` on `microservice2` and merge the results.

#### Rate Limiting

To add rate limiting to protect your microservices, use the `rate-limit` module in the extra\_config section:

```json
json{
  "extra_config": {
    "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
      "maxRate": 10,   // Maximum 10 requests per second
      "capacity": 20   // Bucket size (max burst of 20)
    }
  }
}
```

#### JWT Authentication

KrakenD can enforce authentication with JWTs to secure your API. You need to define this in the `extra_config` section:

```json
json{
  "extra_config": {
    "github.com/devopsfaith/krakend-jose/validator": {
      "alg": "HS256",
      "jwk-url": "https://your-auth-server.com/.well-known/jwks.json",
      "audience": ["your-audience"],
      "issuer": "https://your-issuer.com"
    }
  }
}
```

#### Caching

To reduce the load on your microservices, you can enable response caching:

```json
json{
  "extra_config": {
    "github.com/devopsfaith/krakend-httpcache": {
      "shared": true,
      "max_age": 60  // Cache responses for 60 seconds
    }
  }
}
```

### 4\. **Deploying KrakenD with Docker**

To deploy KrakenD as a Docker container, create a `Dockerfile`:

```Dockerfile
DockerfileFROM devopsfaith/krakend:2.1.1
COPY ./config/krakend.json /etc/krakend/krakend.json
EXPOSE 8080
ENTRYPOINT ["krakend", "run", "-c", "/etc/krakend/krakend.json"]
```

Then, build and run the Docker container:

```bash
bashdocker build -t krakend-gateway .
docker run -p 8080:8080 krakend-gateway
```

### Summary:

* **krakend.json** is the core configuration file defining the API Gateway.
* The **endpoints** section defines the public routes.
* **backend** defines the microservices KrakenD interacts with.
* You can use advanced features like caching, rate limiting, request aggregation, and JWT authentication via the `extra_config` section.
* The folder structure keeps the configuration, plugins, and logs organized.

## The **partials**, **settings**, and **templates** folders
> In KrakenD, the **partials**, **settings**, and **templates** folders are used to modularize and simplify the management of the API gateway configuration. These folders help in breaking down the configuration into smaller, reusable components, which can then be assembled into a full configuration file.

### 1\. **partials Folder**

* **Purpose**: The `partials` folder is used to store reusable configuration blocks. These blocks can be included in the main `krakend.json` file to avoid repetition and improve maintainability.
* **Files**: Each file in the `partials` folder contains a section of the overall KrakenD configuration, such as backend configurations, endpoints, extra config, etc.
* **Usage**: Partial files are usually included into the main config using KrakenD’s [Flexibility Configuration]().

**Example**: A partial file (`partials/users.json`) might look like this:

```json
json{
  "backend": [
    {
      "host": ["http://microservice1:8080"],
      "url_pattern": "/users",
      "encoding": "json"
    }
  ]
}
```

Then, in the main `krakend.json` file, you can include it:

```json
json{
  "$ref": "partials/users.json"
}
```

### 2\. **settings Folder**

* **Purpose**: The `settings` folder stores configuration settings that apply globally or to specific parts of the KrakenD configuration. This can include rate limiting, timeouts, security settings, or any other global setting applicable to various endpoints.
* **Files**: Files in the `settings` folder usually contain configuration parameters or shared configuration blocks that can be referenced throughout the project.

**Example**: You might have a settings file (`settings/global.json`) for global parameters:

```json
json{
  "timeout": "3000ms",
  "cache_ttl": "600s"
}
```

This setting can be included in multiple places in the `krakend.json` file.

### 3\. **templates Folder**

* **Purpose**: The `templates` folder contains templates that allow you to generate dynamic parts of the configuration. These templates can be used to create consistent, reusable structures across different endpoints or services.
* **Files**: Files in the `templates` folder often use variables and placeholders, making it easier to maintain and replicate common structures.

**Example**: Suppose you have a template file (`templates/backend.tmpl`) that contains:

```json
json{
  "host": ["{{ .Host }}"],
  "url_pattern": "{{ .UrlPattern }}",
  "encoding": "json"
}
```

In the main `krakend.json`, you can reference this template and pass in the values for the variables:

```json
json{
  "backend": [
    {
      "$ref": "templates/backend.tmpl",
      "Host": "http://microservice1:8080",
      "UrlPattern": "/users"
    }
  ]
}
```

### 4\. **.tmpl Files**

* **Purpose**: Files with the `.tmpl` extension are used for templates. These templates allow you to inject variables into configurations, making the configuration process more dynamic and maintainable.
* **Usage**: KrakenD uses Go templates for `.tmpl` files. Inside a `.tmpl` file, you can define placeholders using `{{ .VariableName }}` and then inject values when including the template into your configuration.

**Example of a `.tmpl` File**:

```json
json{
  "url_pattern": "{{ .UrlPattern }}",
  "host": ["{{ .Host }}"],
  "method": "{{ .Method }}",
  "encoding": "json"
}
```

When you include this template in your `krakend.json`, you pass the specific values like this:

```json
json{
  "$ref": "templates/backend.tmpl",
  "UrlPattern": "/products",
  "Host": "http://microservice2:8080",
  "Method": "GET"
}
```

### Benefits of Using Partials, Settings, and Templates:

* **Modularization**: By separating the configuration into smaller, reusable blocks, it becomes easier to manage and maintain.
* **Reusability**: Templates and partials can be reused across different endpoints, reducing repetition and making the configuration more DRY (Don’t Repeat Yourself).
* **Consistency**: Common configurations, such as timeouts or rate limits, can be applied uniformly across all services using the settings folder.

### Folder Structure Example:

```bash
bash/krakend-gateway/
├── config/
│   ├── krakend.json              # Main KrakenD configuration file
│   ├── partials/                 # Reusable configuration blocks
│   │   ├── users.json
│   │   └── orders.json
│   ├── settings/                 # Global or shared settings
│   │   └── global.json
│   └── templates/                # Dynamic templates for configuration
│       └── backend.tmpl
└── Dockerfile
```

> By structuring the configuration into these folders, it is easier to manage the complexity of KrakenD configurations, especially in large-scale microservice environments.

