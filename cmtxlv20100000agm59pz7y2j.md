---
title: "Building a CRUD API with FastAPI "
seoTitle: "Building a CRUD API with FastAPI"
seoDescription: "Building a CRUD API with FastAPI"
datePublished: 2026-09-01T09:30:00.000Z
cuid: cmtxlv20100000agm59pz7y2j
slug: building-a-crud-api-with-fastapi
cover: https://cdn.hashnode.com/uploads/covers/6a96addb74971ca68e6a6233/d0945d51-00ce-4997-9efe-ecf63afb7d20.png
tags: python, crud, fastapi, crud-operations, backend-systems

---

### Introduction

When building a backend application, it can be useful to start with the simplest working version before introducing databases, authentication, and more complex architecture. In this tutorial, I begin with an in-memory FastAPI application to implement basic CRUD operations. This makes it easier to understand the request-response cycle, Pydantic models, path parameters, HTTP status codes, and error handling before moving on to persistent storage such as PostgreSQL. In that regard this is part of the series and it focuses on explaining how i am building the first version of that API step by step. By the end, the API will support:

> `POST    /assessments`
> 
> `GET     /assessments`
> 
> `GET     /assessments/{assessment_id}`
> 
> `PATCH   /assessments/{assessment_id}`
> 
> `DELETE  /assessments/{assessment_id}`

The data will initially be stored in a Python list.

1\. Create the FastAPI application

Start by importing FastAPI and creating an application instance.

> `from fastapi import FastAPI`
> 
> `app = FastAPI()`

The app object represents the FastAPI application.

FastAPI uses Python decorators to connect HTTP requests to Python functions.

For example:

> `@app.get("/health")`
> 
> `async def health_check():`
> 
> `return {"status": "OK"}`

Here:

`@app.get("/health")`

means:

When the server receives a GET request for /health, run the function below it.

We can make the health endpoint slightly more descriptive:

> `@app.get("/health")`
> 
> `async def health_check():`
> 
> `return {`
> 
> `"status": "OK",`
> 
> `"service": "Responsibility Lens API"`
> 
> `}`

If the application is running correctly, visiting:

> GET /health
> 
> returns:
> 
> {
> 
> "status": "OK",
> 
> "service": "Responsibility Lens API"
> 
> }

This gives a simple way to confirm that the API is available.

### 2\. Create temporary in-memory storage

The second step is that before adding a database, I used a Python list:

`assessments = []`

Each assessment created by the API will be stored in this list.

Hypothetically from :

`Please note that the phrase "move to", was used figuratively.`

> HTTP request
> 
> *"move to"*
> 
> FastAPI route
> 
> *"move to"*
> 
> Python logic
> 
> *"move to"*
> 
> assessments list
> 
> *"move to"*
> 
> HTTP response

This is useful for learning and early prototyping, but it has an important limitation: *The data disappears whenever the application restarts, which means that every time you run the process you must be aware of that limitation*. Later, this list will be replaced with PostgreSQL to rework that , denoting the significance of adding PostgreSQL.

### 3\. Define the request model with Pydantic

The third step is to define an assessment that needs three pieces of information:

> system\_name
> 
> organisation
> 
> description

Instead of accepting arbitrary JSON, FastAPI can use a Pydantic model to define the expected request structure.

`from pydantic import BaseModel`

Then create the model:

> class AssessmentCreate(BaseModel):
> 
> system\_name: str
> 
> organisation: str
> 
> description: str

Now FastAPI expects a request such as:

> `{`
> 
> `"system_name": "AI Credit Review System",`
> 
> `"organisation": "Example Bank",`
> 
> `"description": "An AI-assisted system used to support credit decisions."`
> 
> `}`

If required fields are missing or contain the wrong data type, FastAPI automatically rejects the request. This is one of the reasons Pydantic is useful in FastAPI applications: it creates a clear contract between the client and the backend.

4\. Create an assessment

The forth step is to create an assesment and by that we create the first POST endpoint.

> @[app.post](http://app.post)("/assessments")
> 
> async def create\_assessment(assessment: AssessmentCreate):
> 
> assessment\_data = assessment.model\_dump()
> 
> assessment\_data\["id"\] = len(assessments) + 1
> 
> assessments.append(assessment\_data)
> 
> return assessment\_data

There are several things happening here.

`Receiving the request assessment:`

`AssessmentCreate tells FastAPI: Parse the incoming request body using the AssessmentCreate model.`

`So FastAPI first validates the JSON before our function runs.`

### 4b.Converting the Pydantic object

The validated JSON object from FastAPi is a Pydantic model. To turn it into a normal Python dictionary we can use the `model_dump()` method:

`assessment_data = assessment.model_dump()`

For example:

> AssessmentCreate(
> 
> system\_name="AI Credit Review System",
> 
> organisation="Example Bank",
> 
> description="AI-assisted credit review."
> 
> )
> 
> becomes approximately:
> 
> {
> 
> "system\_name": "AI Credit Review System",
> 
> "organisation": "Example Bank",
> 
> "description": "AI-assisted credit review."
> 
> }

### 4c. Creating an ID

For the next step because I am still working with the prototype, and yet to have add a database yet, I generated a simple identifier:

> assessment\_data\["id"\] = len(assessments) + 1
> 
> If the list is empty:
> 
> len(assessments) = 0
> 
> the first record receives:
> 
> id = 1

This is sufficient for the prototype, although it is important to note that with a real database, persistent identifiers should be generated.

4d. Saving the assessment

The record is then added to the list:

> assessments.append(assessment\_data)
> 
> and returned to the client: return assessment\_data

### 5.Return the correct HTTP status code

The next step is to create a resource that should normally return:

`201 Created rather than the default: 200 OK`

This is becuase FastAPI provides HTTP status constants:

`from fastapi import status`

We can then update the route:

> @[app.post](http://app.post)(
> 
> "/assessments",
> 
> status\_code=status.HTTP\_201\_CREATED
> 
> async def create\_assessment(assessment: AssessmentCreate):
> 
> assessment\_data = assessment.model\_dump()
> 
> assessment\_data\["id"\] = len(assessments) + 1
> 
> assessments.append(assessment\_data)
> 
> return assessment\_data

Using: `status.HTTP_201_CREATED is more readable than writing: status_code=201`

Both work, but the named constant communicates the meaning of the status code.

### 6\. Retrieve all assessments

At the stage, the next endpoint returns everything stored in the list.

> @app.get("/assessments")
> 
> async def assessments\_list():
> 
> return assessments
> 
> If two assessments have been created, the response could look like:
> 
> \[
> 
> {
> 
> "system\_name": "AI Credit Review System",
> 
> "organisation": "Example Bank",
> 
> "description": "AI-assisted credit review system.",
> 
> "id": 1
> 
> },
> 
> {
> 
> "system\_name": "Clinical Decision Support",
> 
> "organisation": "Example Hospital",
> 
> "description": "AI-assisted clinical recommendation system.",
> 
> "id": 2
> 
> }
> 
> \]

The application flow is now:

> `POST /assessments`
> 
> > `create assessment then store in list`
> 
> `GET /assessments`
> 
> > `read list then return assessments`

**That brings an end to the Part One of *Building a CRUD API with FastAPI: From an In-Memory Prototype to a Working Backend*.**

In this part, we set up the FastAPI application, created our assessment model, and implemented the first CRUD operations for creating and retrieving assessments. In **Part Two**, we’ll complete the CRUD cycle by implementing **update and delete operations**, while looking more closely at partial updates, path parameters, and error handling.