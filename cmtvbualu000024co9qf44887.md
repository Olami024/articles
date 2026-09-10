---
title: "How I Refactored a Streamlit Prototype into a Modular Python Application"
seoTitle: "Streamlit Prototype into a Modular Python Application"
seoDescription: "What building a real project taught me about separation of concerns, testable business logic, and designing a prototype for what comes next."
datePublished: 2026-09-10T09:30:00.000Z
cuid: cmtvbualu000024co9qf44887
slug: how-i-refactored-a-streamlit-prototype-into-a-modular-python-application
cover: https://cdn.hashnode.com/uploads/covers/6a96addb74971ca68e6a6233/888eab3e-1114-4714-a676-d0e3987f7c5b.jpg
tags: python, database, modularity, applications, product, decision-making, designstrategy

---

When I started building the **Human Judgment Preservation Index (HJPI)**, the first proprietary methodology within **Responsibility Lens**, my priority was simple: make the idea work.

Responsibility Lens is a product I am building around **decision responsibility**. The **Human Judgment Preservation Index (HJPI)** focuses on how AI-assisted systems may affect human judgment, agency, responsibility, and meaningful oversight. The first working version was built with **Python, Streamlit, and the logical rules behind the methodology**.

Streamlit was useful at that stage because it allowed me to move quickly from an idea and methodology to something interactive. Users could respond to assessment questions, receive calculated results, view radar charts, and download results as CSV files. At this stage, the application did not yet have a database or API.

It worked. But as I continued learning, reviewing the project, and thinking about how Responsibility Lens might develop, one problem became increasingly obvious: **too many parts of the application lived in the same place.**

The application worked, but the structure underneath it was becoming harder to reason about. That was when I decided to refactor it. The first version of Responsibility Lens grew naturally around the Streamlit interface. That meant the application gradually became responsible for several different things at once:

1.  rendering the user interface;
    
2.  representing the HJPI methodology;
    
3.  calculating scores;
    
4.  storing configuration;
    
5.  generating charts;
    
6.  preparing exports;
    
7.  displaying explanations and results.
    

This convenience at one stage can become coupling at another and as  the application develops, it becomes harder to distinguish between **what the product was doing** and **how Streamlit was displaying it**.

For example, scoring logic should be concerned with a question such as: Given these assessment responses, what result should the methodology produce? It does not need to know which Streamlit component collected those responses. In the same way, the methodology should describe the dimensions, questions, options, and related information used by the assessment. It does not need to know how a particular page is arranged. These distinctions sound obvious when written down. They were much less obvious while I was focused on getting the first prototype to work.

###   
Why I decided to refactor

Responsibility Lens is still using Streamlit, so the goal of the refactor was not to remove Streamlit. Instead, I started asking a different question: What happens if Responsibility Lens grows beyond this interface? That changed how I thought about the codebase. I identified below the four main reasons while I wanted to restructure the app.

1.  Maintainability: I wanted different responsibilities to have clearer places in the application. That makes basic development questions easier to answer: Where does this calculation happen? Where is the methodology defined? Where should I modify a chart? Where does export formatting belong? Which part of the application owns this behaviour? A clearer structure also reduces the amount of unrelated code I need to think about when making one change.
    
2.  Testing: Another reason is that business logic embedded inside UI code is difficult to test independently. I wanted to be able to give a scoring function to a known set of inputs, run it directly, and inspect the result without starting Streamlit or manually clicking through the application.That meant the scoring logic needed to exist independently of the interface.
    
3.  Reuse: Responsibility Lens may not always have only one interface. The underlying methodology may eventually need to support an API, another interface, reports, or other services. If the core logic depends directly on Streamlit, reusing that logic elsewhere becomes unnecessarily difficult.
    
4.  Preparing for backend development: I am building Responsibility Lens progressively, so while Streamlit remains useful for the current stage of the project, I am also preparing the application for a more conventional backend architecture using technologies such as FastAPI and PostgreSQL. Because of that, modularity became more than a code-cleanliness exercise. It became part of preparing the project for its next stage.
    

### **Moving toward a modular Python structure**

With the above realization, the next step was to create a cleaner structure for the HJPI application. A simplified version looks like this:

`hjpi/`

`├──` [`methodology.py`](http://methodology.py)

`├──` [`config.py`](http://config.py)

`├──` [`scoring.py`](http://scoring.py)

`├──` [`charts.py`](http://charts.py)

`└──` [`exports.py`](http://exports.py)

`hjpi_`[`app.py`](http://app.py)

  
Each module now has a clearer responsibility.

[methodology.py](http://methodology.py) contains the structures that describe the assessment methodology.

[config.py](http://config.py) contains configuration that should not be scattered throughout the application.

[scoring.py](http://scoring.py) contains the logic responsible for calculations.

[charts.py](http://charts.py) handles visualisation-related functionality.

[exports.py](http://exports.py) handles the generation of exportable results.

And hjpi\_[app.py](http://app.py) remains responsible for the Streamlit application itself.

This is not a complicated architecture. That is intentional. As I was not trying to introduce abstractions simply because I could. I wanted the structure of the code to reflect responsibilities that had already emerged naturally while building the product.

## **Separating business logic from Streamlit**

One of the most important changes was separating **business logic from interface logic**.

Consider a simplified scoring example:

`def calculate_score(responses, weights):`

    `weighted_total = sum(`

        `responses[key] * weights[key]`

        `for key in responses`

    `)`

    `total_weight = sum(`

        `weights[key]`

        `for key in responses`

    `)`

    `return weighted_total / total_weight`

  
  
This is only a simplified illustration. The actual HJPI methodology contains its own domain-specific rules. What matters here is the architectural principle. The function does not import Streamlit. It does not display a message. It does not create a chart. It does not need to know whether the responses came from a radio button, an API request, a database record, or a test. It receives data, performs a calculation, and returns a result.

The Streamlit application can then use that result:

`score = calculate_score(responses, weights)`

`st.metric(`

    `label="Assessment Score",`

    `value=score`

`)`

This creates a clearer boundary of moving from interface through to application logic and then result. That separation gives the scoring logic a life outside Streamlit. If I later expose parts of the assessment through FastAPI, the underlying logic should not need to be rewritten simply because the interface has changed. That was one of the most important conceptual shifts in the refactor.

## **Making the methodology data-driven**

Another area I reconsidered was how the methodology itself was represented.

HJPI contains structured domain information. As the methodology develops, I do not want every new or revised dimension, question, or description to require another manually written block of interface code. As Responsibility Lens develops, some of the areas I want tests around include:

*   expected scoring behaviour;
    
*   boundary conditions;
    
*   missing or invalid inputs;
    
*   methodology configuration;
    
*   calculations used in results;
    
*   transformations used for exports.
    

The refactor does not automatically make the application reliable. What it does is make reliability easier to build. **The refactor changed how I understood the project.** One thing I did not expect was how much restructuring the code would change the way I thought about Responsibility Lens itself. Initially, I largely thought of what I had built as a Streamlit application. After refactoring, I could see the different parts of the system more clearly:`   `

`Responsibility Lens`

        `│`

        `├── Methodology`

        `│`

        `├── Domain logic`

        `│`

        `├── Application logic`

        `│`

        `├── Presentation`

        `│`

        `├── Reporting`

        `│`

        `└── Infrastructure`

  
That distinction gave the project more meaning for me. Responsibility Lens became less like **a Streamlit app** and more like **a system whose current interface happens to be Streamlit**. That is an important difference. Streamlit is one way of interacting with the system. It does not have to define the system itself. And that has influenced how I am thinking about the next stages of the project. **What I learned from the refactor** The biggest lesson was not that every prototype needs an elaborate architecture from the beginning. I still think there is value in building the first version quickly enough to understand the problem. But once different responsibilities begin to emerge, it becomes important to recognise them. I learned that scoring logic should be able to exist without the interface.

In essence, the refactor helped me begin answering another: **How should I structure this application if I want to continue building it?** Responsibility Lens is still developing, and so is my understanding of backend engineering. But moving the HJPI application from a tightly coupled Streamlit prototype toward a more modular Python structure was an important step. I started with something that worked. Now I am learning how to build something that can continue to evolve.