---
title: "Setting Up Claude Code With Ollama: A Guide"
source: "https://dzone.com/articles/claude-code-ollama-setup-guide"
author:
  - "[[Gunter Rotsaert]]"
published: 2026-05-05
created: 2026-05-06
description: "Claude Code is a terminal-based AI coding assistant by Anthropic that can integrate with Ollama for local model inference."
tags:
  - "clippings llm claud code"
---
Join the DZone community and get the full member experience.

[Join For Free](https://dzone.com/static/registration.html)

Nowadays, there are quite a lot of AI coding assistants. In this blog, you will take a closer look at Claude Code, a terminal-based AI coding assistant. Since mid January 2026, Claude Code can also be used in combination with Ollama, a local inference engine. Enjoy!

## Introduction

There are many AI models and also many AI coding assistants. Which one to choose is a hard question. It also depends on whether you run the models locally or in the cloud. When running locally, Qwen3-Coder is a very good AI model to be used for programming tasks. In previous posts, [DevoxxGenie](https://mydeveloperplanet.com/2024/10/08/devoxxgenie-your-ai-assistant-for-idea/), a JetBrains IDE plugin, was often used as an AI coding assistant. DevoxxGenie is nicely integrated within the JetBrains IDEs. But it is also a good thing to take a look at other AI coding assistants. In a [previous blog](https://mydeveloperplanet.com/2026/02/25/getting-started-with-qwen-code-for-coding-tasks/), Qwen Code was used; now it is time to take a look at Claude Code.

Claude Code is the popular AI coding assistant from Anthropic. Since [mid January 2026](https://ollama.com/blog/claude), Claude Code can be used in combination with Ollama, a local inference engine. Using local inference engines, you are 100% sure your data is not shared with third parties like Anthropic. In contrast to Qwen Code, which is based on Gemini CLI, Claude Code is not entirely open source; the core executable is closed source.

In this blog, you will take a closer look at Claude Code, how to configure it, and how to use it.

Sources used in this blog can be found on [GitHub](https://github.com/mydeveloperplanet/myaicodeprojectplanet/tree/claude-code).

## Prerequisites

Prerequisites for reading this blog are:

- Some experience with AI coding assistants.
- If you want to compare to DevoxxGenie, take a look at a [previous post](https://mydeveloperplanet.com/2024/10/08/devoxxgenie-your-ai-assistant-for-idea/).
- You will need to have at least Ollama v0.14.0 installed.

## Installation

Installation instructions for Claude Code can be found [here](https://github.com/anthropics/claude-code).

Execute the following bash script.

Shell

```text/x-sh
curl -fsSL https://claude.ai/install.sh | bash
```

## Setup

Claude Code is installed now, but first, some configuration needs to be done. See also the [official documentation](https://code.claude.com/docs) for Claude Code.

### 1\. Disable Data Usage

By default, Claude Code is monitoring [data usage](https://code.claude.com/docs/en/data-usage). It is not very clear whether this applies only when you use cloud models or cloud APIs. Nevertheless, it is advised to disable this by means of an environment variable. You can configure settings at different [scopes](https://code.claude.com/docs/en/settings#configuration-scopes): there is a managed, user, project, and local scope. For convenience, the user scope is used here.

Navigate to your home directory, and you will see a **.claude** directory. Create a file **settings.json** in this **.claude** directory. Disable the usage statistics. A full list of the environment variables can be found [here](https://code.claude.com/docs/en/settings#environment-variables).

JSON

```application/json
{
```

```application/json
"env": {
```

```application/json
"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
```

```application/json
}
```

```application/json
}
```

Do note that this will also disable the auto-updates. In order to avoid this, you can disable all other traffic environment variables.

JSON

7

```application/json
{
```

```application/json
"env": {
```

```application/json
"DISABLE_TELEMETRY": "1",
```

```application/json
"DISABLE_BUG_COMMAND": "1",
```

```application/json
"DISABLE_ERROR_REPORTING": "1"
```

```application/json
}
```

```application/json
}
```

### 2\. Configure Model

In this blog, a local model setup is used, using Ollama as the inference engine and Qwen3-Coder running as a local model. In order to create the setup for this, add the following to the **settings.json** file.

JSON

8

```application/json
{
```

```application/json
"env": {
```

```application/json
"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
```

```application/json
"ANTHROPIC_AUTH_TOKEN": "ollama",
```

```application/json
"ANTHROPIC_API_KEY": "",
```

```application/json
"ANTHROPIC_BASE_URL": "http://localhost:11434"
```

```application/json
}
```

```application/json
}
```

And if you want to set the default model, you can do so as follows.

JSON

9

```application/json
{
```

```application/json
"model": "qwen3-coder:30b",
```

```application/json
"env": {
```

```application/json
"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
```

```application/json
"ANTHROPIC_AUTH_TOKEN": "ollama",
```

```application/json
"ANTHROPIC_API_KEY": "",
```

```application/json
"ANTHROPIC_BASE_URL": "http://localhost:11434"
```

```application/json
}
```

```application/json
}
```

### 3\. System Prompt

It is a good practice to add a system prompt to your AI coding assistant. You can add some instructions for the model in this system prompt. You can add it by creating a **CLAUDE.md** file in the **.claude** directory. This will ensure a default system prompt for all your projects. If you want to use a more specific system prompt for a particular repository, you can add a **CLAUDE.md** file in the repository itself.

If you are developing in Java, Spring Boot, etc., the following system prompt can be used as an example.

```text/plain
You are an expert code assistant for a professional Java developer. All code examples, reviews, and explanations must be idiomatic to the following tech stack:
```

```text/plain
* Backend: Java (latest LTS), Spring Boot (latest stable), PostgreSQL.
```

```text/plain
* Frontend: Vue.js (latest stable), Angular (latest stable).
```

```text/plain
* Follow modern best practices for RESTful APIs, object-relational mapping, unit testing (JUnit), and frontend-backend integration.
```

```text/plain
* Prefer Maven for Java dependency management.
```

```text/plain
* Whenever database code is required, use PostgreSQL syntax and conventions.
```

```text/plain
* For frontend, use Vue composition API where applicable.
```

```text/plain
* Always explain your reasoning, and reference documentation when giving architectural advice.
```

```text/plain
* When unsure, ask clarifying questions before producing code.
```

### 4\. Default Editor

The default editor seems to be Visual Studio Code, but you can change this. In order to change it, you need to override the **EDITOR** environment variable. In the example below, it is set to **vim**. Add this to your **.bashrc** file.

Shell

```text/x-sh
export EDITOR='vim'
```

### 5\. Fix Slow Inference

Claude Code prepends and adds a Claude Code Attribution header, which invalidates the KV Cache, making inference 90% slower with local models. See [this article](https://unsloth.ai/docs/basics/claude-code#fixing-90-slower-inference-in-claude-code) for more information. You can solve this by setting the **CLAUDE\_CODE\_ATTRIBUTION\_HEADER** to zero in the **settings.json**.

JSON

10

```application/json
{
```

```application/json
"model": "qwen3-coder:30b",
```

```application/json
"env": {
```

```application/json
"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
```

```application/json
"CLAUDE_CODE_ATTRIBUTION_HEADER" : "0",
```

```application/json
"ANTHROPIC_AUTH_TOKEN": "ollama",
```

```application/json
"ANTHROPIC_API_KEY": "",
```

```application/json
"ANTHROPIC_BASE_URL": "http://localhost:11434"
```

```application/json
}
```

```application/json
}
```

## First Startup

If you haven't done it already, now is the time to clone the [GitHub](https://github.com/mydeveloperplanet/myaicodeprojectplanet/tree/claude-code) repository. Be sure to check out the **c** **laude-code** branch. If you want to execute the commands from this blog, you first delete the **CLAUDE.md** file and the **src/test** directory.

Claude Code is a terminal application, so you have some different options here:

- Open a terminal and navigate to the repository.
- Open your IDE, e.g., IntelliJ, and open a terminal from within IntelliJ (ALT+F12).

Do note that using the IntelliJ terminal will react differently than expected to certain commands. It is advised to use a regular terminal instead.

Start Claude Code by typing `claude` in the terminal. The first time you start Claude Code in a repository, it will ask you whether you trust this folder.

![Claude Code will ask you whether you trust this folder](https://dz2cdn1.dzone.com/storage/temp/19009037-claude-code-first-startup-do-you-trust-folder.png)

Claude Code will ask you whether you trust this folder

After confirmation, the Claude prompt is shown.

![Claude prompt](https://dz2cdn1.dzone.com/storage/temp/19009038-claude-code-first-startup-cli.png)

Claude prompt

A first simple command is to show the model that is loaded. As expected, Qwen3-Coder is loaded.

![Qwen3-Coder is loaded](https://dz2cdn1.dzone.com/storage/temp/19009039-claude-code-first-startup-check-selected-model.png)

Qwen3-Coder is loaded

When you want to show the memory content, you are asked to show the user or project settings. When you choose a **user**, the **CLAUDE.md** file is opened.

Now, in order to verify whether the connection with the model is functioning correctly, just enter a simple prompt like **how are you?** The response is the following:

26

```text/plain
● Hello! I'm functioning properly and ready to help with your Java development tasks. I'm set up to assist with Spring Boot backend development, PostgreSQL database integration, and Vue.js/Angular frontend work as
```

```text/plain
specified in your preferences.
```

```text/plain
Is there something specific you'd like me to help you with today? For example, I can help with:
```

```text/plain
- Spring Boot application development
```

```text/plain
- PostgreSQL database queries or schema design
```

```text/plain
- Java code reviews
```

```text/plain
- Vue.js/Angular frontend integration
```

```text/plain
- RESTful API design
```

```text/plain
- Unit testing with JUnit
```

```text/plain
What would you like to work on?
```

```text/plain
No tasks found
```

```text/plain
● I don't see any existing tasks in the task list. Would you like me to help you with a specific Java development task? I'm ready to assist with Spring Boot backend development, PostgreSQL database work, or
```

```text/plain
Vue.js/Angular frontend integration as per your preferences.
```

```text/plain
You can ask me to:
```

```text/plain
- Review or improve Java code
```

```text/plain
- Help with Spring Boot application development
```

```text/plain
- Assist with PostgreSQL database queries or schema design
```

```text/plain
- Help with RESTful API implementation
```

```text/plain
- Assist with unit testing using JUnit
```

```text/plain
- Help with frontend-backend integration
```

```text/plain
What would you like to work on today?
```

## Create a Test

Let's continue with something useful and create a test for the `CustomersController`.

Using the **@** character, you can add files to the context. When typing, a search is executed, and using the arrows, you can easily select the file you need. Using the tab, you select the file. After that, you can complete the prompt. The prompt used is:

Java

x

```text/x-java
@src/main/java/com/mydeveloperplanet/myaicodeprojectplanet/controller/CustomersController.java
```

```text/x-java
​
```

```text/x-java
Write a unit test for this code using JUnit.
```

```text/x-java
Use WebMvcTest.
```

```text/x-java
Use MockMvc.
```

```text/x-java
Use AssertJ assertions.
```

```text/x-java
​
```

```text/x-java
Add the test in this repository
```

Claude Code starts analyzing the file and writes the test.

Claude Code will ask you whether it may execute **bash**.

The test created can be seen below. Just like with Qwen Coder, a similar error occurs. A type OpenAPICustomer is used, which does not exist.

Java

136

```text/x-java
package com.mydeveloperplanet.myaicodeprojectplanet.controller;
```

```text/x-java
​
```

```text/x-java
import com.mydeveloperplanet.myaicodeprojectplanet.model.Customer;
```

```text/x-java
import com.mydeveloperplanet.myaicodeprojectplanet.openapi.model.Customer;
```

```text/x-java
import com.mydeveloperplanet.myaicodeprojectplanet.service.CustomerService;
```

```text/x-java
import org.junit.jupiter.api.BeforeEach;
```

```text/x-java
import org.junit.jupiter.api.Test;
```

```text/x-java
import org.springframework.beans.factory.annotation.Autowired;
```

```text/x-java
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
```

```text/x-java
import org.springframework.boot.test.mock.mockito.MockBean;
```

```text/x-java
import org.springframework.http.MediaType;
```

```text/x-java
import org.springframework.test.web.servlet.MockMvc;
```

```text/x-java
​
```

```text/x-java
import java.util.Arrays;
```

```text/x-java
import java.util.List;
```

```text/x-java
import java.util.Optional;
```

```text/x-java
​
```

```text/x-java
import static org.mockito.ArgumentMatchers.any;
```

```text/x-java
import static org.mockito.Mockito.*;
```

```text/x-java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
```

```text/x-java
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
```

```text/x-java
​
```

```text/x-java
@WebMvcTest(controllers = CustomersController.class)
```

```text/x-java
class CustomersControllerTest {
```

```text/x-java
​
```

```text/x-java
@Autowired
```

```text/x-java
private MockMvc mockMvc;
```

```text/x-java
​
```

```text/x-java
@MockBean
```

```text/x-java
private CustomerService customerService;
```

```text/x-java
​
```

```text/x-java
private Customer customer1;
```

```text/x-java
private Customer customer2;
```

```text/x-java
private OpenAPICustomer openAPICustomer1;
```

```text/x-java
private OpenAPICustomer openAPICustomer2;
```

```text/x-java
​
```

```text/x-java
@BeforeEach
```

```text/x-java
void setUp() {
```

```text/x-java
customer1 = new Customer(1L, "John", "Doe");
```

```text/x-java
customer2 = new Customer(2L, "Jane", "Smith");
```

```text/x-java
​
```

```text/x-java
openAPICustomer1 = new OpenAPICustomer()
```

```text/x-java
.id(1L)
```

```text/x-java
.firstName("John")
```

```text/x-java
.lastName("Doe");
```

```text/x-java
​
```

```text/x-java
openAPICustomer2 = new OpenAPICustomer()
```

```text/x-java
.id(2L)
```

```text/x-java
.firstName("Jane")
```

```text/x-java
.lastName("Smith");
```

```text/x-java
}
```

```text/x-java
​
```

```text/x-java
@Test
```

```text/x-java
void customersGet_shouldReturnAllCustomers() throws Exception {
```

```text/x-java
// Given
```

```text/x-java
List<Customer> customers = Arrays.asList(customer1, customer2);
```

```text/x-java
when(customerService.getAllCustomers()).thenReturn(customers);
```

```text/x-java
​
```

```text/x-java
// When & Then
```

```text/x-java
mockMvc.perform(get("/customers"))
```

```text/x-java
.andExpect(status().isOk())
```

```text/x-java
.andExpect(jsonPath("$.length()").value(2))
```

```text/x-java
.andExpect(jsonPath("$[0].id").value(1L))
```

```text/x-java
.andExpect(jsonPath("$[0].firstName").value("John"))
```

```text/x-java
.andExpect(jsonPath("$[0].lastName").value("Doe"))
```

```text/x-java
.andExpect(jsonPath("$[1].id").value(2L))
```

```text/x-java
.andExpect(jsonPath("$[1].firstName").value("Jane"))
```

```text/x-java
.andExpect(jsonPath("$[1].lastName").value("Smith"));
```

```text/x-java
​
```

```text/x-java
verify(customerService, times(1)).getAllCustomers();
```

```text/x-java
}
```

```text/x-java
​
```

```text/x-java
@Test
```

```text/x-java
void customersPost_shouldCreateNewCustomer() throws Exception {
```

```text/x-java
// Given
```

```text/x-java
when(customerService.createCustomer(any(Customer.class))).thenReturn(customer1);
```

```text/x-java
​
```

```text/x-java
// When & Then
```

```text/x-java
mockMvc.perform(post("/customers")
```

```text/x-java
.contentType(MediaType.APPLICATION_JSON)
```

```text/x-java
.content("{\"firstName\":\"John\",\"lastName\":\"Doe\"}"))
```

```text/x-java
.andExpect(status().isCreated());
```

```text/x-java
​
```

```text/x-java
verify(customerService, times(1)).createCustomer(any(Customer.class));
```

```text/x-java
}
```

```text/x-java
​
```

```text/x-java
@Test
```

```text/x-java
void customersIdGet_shouldReturnCustomerWhenFound() throws Exception {
```

```text/x-java
// Given
```

```text/x-java
when(customerService.getCustomerById(1L)).thenReturn(Optional.of(customer1));
```

```text/x-java
​
```

```text/x-java
// When & Then
```

```text/x-java
mockMvc.perform(get("/customers/{id}", 1L))
```

```text/x-java
.andExpect(status().isOk())
```

```text/x-java
.andExpect(jsonPath("$.id").value(1L))
```

```text/x-java
.andExpect(jsonPath("$.firstName").value("John"))
```

```text/x-java
.andExpect(jsonPath("$.lastName").value("Doe"));
```

```text/x-java
​
```

```text/x-java
verify(customerService, times(1)).getCustomerById(1L);
```

```text/x-java
}
```

```text/x-java
​
```

```text/x-java
@Test
```

```text/x-java
void customersIdGet_shouldReturnNotFoundWhenCustomerNotFound() throws Exception {
```

```text/x-java
// Given
```

```text/x-java
when(customerService.getCustomerById(1L)).thenReturn(Optional.empty());
```

```text/x-java
​
```

```text/x-java
// When & Then
```

```text/x-java
mockMvc.perform(get("/customers/{id}", 1L))
```

```text/x-java
.andExpect(status().isNotFound());
```

```text/x-java
​
```

```text/x-java
verify(customerService, times(1)).getCustomerById(1L);
```

```text/x-java
}
```

```text/x-java
​
```

```text/x-java
@Test
```

```text/x-java
void customersIdPut_shouldUpdateCustomer() throws Exception {
```

```text/x-java
// Given
```

```text/x-java
when(customerService.updateCustomer(anyLong(), any(Customer.class))).thenReturn(customer1);
```

```text/x-java
​
```

```text/x-java
// When & Then
```

```text/x-java
mockMvc.perform(put("/customers/{id}", 1L)
```

```text/x-java
.contentType(MediaType.APPLICATION_JSON)
```

```text/x-java
.content("{\"firstName\":\"John\",\"lastName\":\"Doe\"}"))
```

```text/x-java
.andExpect(status().isOk());
```

```text/x-java
​
```

```text/x-java
verify(customerService, times(1)).updateCustomer(anyLong(), any(Customer.class));
```

```text/x-java
}
```

```text/x-java
​
```

```text/x-java
@Test
```

```text/x-java
void customersIdDelete_shouldDeleteCustomer() throws Exception {
```

```text/x-java
// When & Then
```

```text/x-java
mockMvc.perform(delete("/customers/{id}", 1L))
```

```text/x-java
.andExpect(status().isNoContent());
```

```text/x-java
​
```

```text/x-java
verify(customerService, times(1)).deleteCustomer(1L);
```

```text/x-java
}
```

```text/x-java
}
```

When you fix the import issue, the test is successful. When you generate the mutation test results using `mvn verify` and check the report in the **target/pit-reports** directory, you notice that this test has a line coverage of 100% and a mutation coverage of 93%, which is quite good. Generating this test with the Claude models of Anthropic results in similar numbers. The generated test using Qwen Coder also has similar results.

## Commands

Claude Code supports [commands](https://code.claude.com/docs/en/interactive-mode#built-in-commands). This is a convenient way of interacting with the model.

Command `/clear` clears the history.

When using the `/init` command, Claude Code analyses your repository and creates a **CLAUDE.md** file in your repository with project-specific information. Executing this command for this repository results in the following **CLAUDE.md** file. The result is really good.

Markdown

75

```text/x-markdown
​
```

```text/x-markdown
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
This is a Spring Boot 3.5 application that implements a RESTful API for managing customer data. The application uses:
```

```text/x-markdown
​
```

```text/x-markdown
- Spring Boot 3.5 with Java 21
```

```text/x-markdown
- PostgreSQL database with Liquibase for database migrations
```

```text/x-markdown
- JOOQ for database access
```

```text/x-markdown
- OpenAPI/Swagger for API documentation
```

```text/x-markdown
- Maven for build management
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
The application follows a layered architecture pattern:
```

```text/x-markdown
​
```

```text/x-markdown
1. **Controller Layer**: REST endpoints in \`CustomersController\`
```

```text/x-markdown
2. **Service Layer**: Business logic in \`CustomerService\` and \`CustomerServiceImpl\`
```

```text/x-markdown
3. **Repository Layer**: Database access in \`CustomerRepository\` using JOOQ
```

```text/x-markdown
4. **Model Layer**: Domain objects in \`Customer\` class
```

```text/x-markdown
5. **OpenAPI Layer**: Generated API interfaces and models from OpenAPI spec
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
- \`src/main/java/com/mydeveloperplanet/myaicodeprojectplanet/MyAiCodeProjectPlanetApplication.java\` - Main application class
```

```text/x-markdown
- \`src/main/java/com/mydeveloperplanet/myaicodeprojectplanet/controller/CustomersController.java\` - REST endpoints
```

```text/x-markdown
- \`src/main/java/com/mydeveloperplanet/myaicodeprojectplanet/service/CustomerServiceImpl.java\` - Business logic
```

```text/x-markdown
- \`src/main/java/com/mydeveloperplanet/myaicodeprojectplanet/repository/CustomerRepository.java\` - Database operations
```

```text/x-markdown
- \`src/main/resources/db/changelog/migration/db.changelog-1.xml\` - Database schema definition
```

```text/x-markdown
- \`src/main/resources/static/customers.yaml\` - OpenAPI specification
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
- \`./mvnw clean compile\` - Compile the application
```

```text/x-markdown
- \`./mvnw clean package\` - Build a jar file
```

```text/x-markdown
- \`./mvnw clean install\` - Install dependencies and build
```

```text/x-markdown
​
```

```text/x-markdown
- \`./mvnw test\` - Run all tests
```

```text/x-markdown
- \`./mvnw test -Dtest=CustomersControllerTest\` - Run specific test class
```

```text/x-markdown
- \`./mvnw test -Dtest=CustomersControllerTest#customersGet_shouldReturnAllCustomers\` - Run specific test method
```

```text/x-markdown
​
```

```text/x-markdown
- \`./mvnw spring-boot:run\` - Run the application
```

```text/x-markdown
- \`./mvnw spring-boot:run -Dspring-boot.run.profiles=dev\` - Run with specific profile
```

```text/x-markdown
​
```

```text/x-markdown
- \`./mvnw generate-sources\` - Regenerate JOOQ and OpenAPI code
```

```text/x-markdown
- \`./mvnw compile\` - Compile with generated code
```

```text/x-markdown
​
```

```text/x-markdown
- \`./mvnw org.pitest:pitest-maven:mutationCoverage\` - Run mutation tests
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
The application uses Liquibase for database migrations. The database schema is defined in \`src/main/resources/db/changelog/migration/db.changelog-1.xml\`. The application will automatically create the database schema on startup.
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
The API provides standard CRUD operations for customers:
```

```text/x-markdown
- GET \`/customers\` - Get all customers
```

```text/x-markdown
- POST \`/customers\` - Create a new customer
```

```text/x-markdown
- GET \`/customers/{id}\` - Get a specific customer
```

```text/x-markdown
- PUT \`/customers/{id}\` - Update a customer
```

```text/x-markdown
- DELETE \`/customers/{id}\` - Delete a customer
```

```text/x-markdown
​
```

```text/x-markdown
​
```

```text/x-markdown
1. The application uses JOOQ for type-safe database access
```

```text/x-markdown
2. OpenAPI specification is used to generate API interfaces and models
```

```text/x-markdown
3. The application uses Spring Boot's auto-configuration for database setup
```

```text/x-markdown
4. Tests are written using Spring Boot's test framework with MockMvc for web layer testing
```

A really nice feature is the option to create custom commands with predefined prompts. Very useful when you want to use prompts repetitively, and you can share them easily with someone else.

Create in the **.claude** directory of your home directory a directory **commands**. Using extra directories inside this **commands** directory, you can create namespaces. As an example, the following directory tree.

Shell

12

```text/x-sh
$ tree
```

```text/x-sh
├── general
```

```text/x-sh
│   ├── explain.md
```

```text/x-sh
│   └── javadoc.md
```

```text/x-sh
├── review
```

```text/x-sh
│   ├── extended.md
```

```text/x-sh
│   └── simple.md
```

```text/x-sh
└── test
```

```text/x-sh
├── controller.md
```

```text/x-sh
├── integration.md
```

```text/x-sh
├── repositoryjooq.md
```

```text/x-sh
└── service.md
```

The **controller.md** file contains the prompt you used for creating the test.

```text/plain
Write a unit test for this code using JUnit.
```

```text/plain
Use WebMvcTest.
```

```text/plain
Use MockMvc.
```

```text/plain
Use AssertJ assertions.
```

## MCP

With [Model Context Protocol](https://qwenlm.github.io/qwen-code-docs/en/users/features/mcp/) (MCP) servers, you can enhance the capabilities of the model.

The [configuration of an MCP server](https://code.claude.com/docs/en/mcp) can be added to the **.claude.json** file in your home directory. The [Context7](https://github.com/upstash/context7) MCP server can be added as follows.

JSON

8

```application/json
"mcpServers": {
```

```application/json
"context7": {
```

```application/json
"type": "stdio",
```

```application/json
"command": "npx",
```

```application/json
"args": ["-y", "@upstash/context7-mcp"],
```

```application/json
"env": {}
```

```application/json
}
```

```application/json
}
```

Start Claude Code in the repository and verify whether the MCP server is configured correctly.

9

```text/plain
/mcp
```

```text/plain
───────────────────────────────────────────────────────────────────
```

```text/plain
Manage MCP servers
```

```text/plain
1 server
```

```text/plain
User MCPs (/home/<userdir>/.claude.json)
```

```text/plain
❯ context7 · ✔ connected
```

```text/plain
https://code.claude.com/docs/en/mcp for help
```

Remove the previously created test, add the `CustomersController` and use the following prompt.

```text/plain
@src/main/java/com/mydeveloperplanet/myaicodeprojectplanet/controller/CustomersController.java /test:controller create the test in this repository, I am using spring boot 3.5, use the context7 mcp server to retrieve
```

```text/plain
uptodate documentation, do not use deprecated functionality
```

This should result in invoking the Context7 MCP server, and as a result `MockBean` should not be used in the test, but `MockitoBean` should be used instead.

However, the Context7 MCP server is invoked, but still the test is generated with `@MockBean`. When you check the [source](https://context7.com/spring-projects/spring-boot/llms.txt) of Context7, which is consulted, you will notice that neither `@MockBean` or `@MockitoBean` is mentioned. This works already better than Qwen Coder, where the Context7 MCP server was not invoked at all, even after many attempts.

## Conclusion

Claude Code offers quite a few nice features. There is a lot more to discover, but the first impressions are good. It is also good to experiment with other AI coding assistants now and then, in order to see how they compare to the ones you are using. Compared to Qwen Coder, Claude Code seems to do a better job.

AI API Component Object Model Spring Boot

Published at DZone with permission of Gunter Rotsaert. [See the original article here.](https://mydeveloperplanet.com/2026/03/18/setting-up-claude-code-with-ollama-a-guide/)

Opinions expressed by DZone contributors are their own.