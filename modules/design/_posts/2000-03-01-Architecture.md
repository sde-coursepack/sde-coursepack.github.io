---
Title: Under Construction
---

* TOC
{:toc}

# Architecture

In software, architecture generally refers to the design of our overall project. So far, in our design unit, we have largely focused on the design at the level of individual classes and methods, or **modules** interacting. With architecture, we want to think about **components**.

## Components

__Components__ describe the major sub-systems in our software system. Typically, for programs you have written, the program is one component. However, if your program is, say, reading from several resource files (such as configuration files, image files, etc.), those files are another __component__: specifically the *data source*. And *where* that file is stored and *how and when* it is accessed in your program is an important decision.

In software architecture, we often think of a component as:
- Groups of modules that handle the same major feature or feature-set
  - For example, in a hotel, they might have a single software system for tracking room vacancy, providing or uploading to an API for room availability to websites booking websites, tracking hours and work tasks (such as room cleanings). These features have intersections, but are most likely separate sub-systems.
- Independent programs (that is, external programs we use)
- Data sources (such as databases, files, webservices, etc.)
- External internet connections, such as servers, browsers, etc.

Very specifically, when talking about components, we do not define modularity as narrowly as we do when talking about individual modules/classes, that is, by the single responsibility principle. However, that doesn't mean there aren't architectural principles to consider.

## Architectural Concerns

When describing software architecture, we want to think about:

1) What are the boundaries of our components where they have to communicate/interact with other components?  
2) How are we separating the major concerns within our system?  
3) How does our architecture affect long term maintainability?  

### Boundaries

Your software is almost certain to interact with systems you have no control over. This can be as simple as using third-party library, like poi or json, but it can also be related to other software you interact with. For example, in earlier examples, I used the [balldontlie.io](https://www.balldontlie.io/#introduction) API to interact with [NBA Team Data](https://github.com/sde-coursepack/NBAExcelTeams). However, baked within that work I had to write code that, on it's face, looks pretty **brittle**.

```java
    private NBATeam getTeamFromJSONObject(JSONObject teamJSon) {
        int id = teamJSon.getInt("id");
        String abbreviation = teamJSon.getString("abbreviation");
        String city = teamJSon.getString("city");
        String name = teamJSon.getString("name");

        Conference conference = getConference(teamJSon);
        Division division = getDivision(teamJSon);

        return new NBATeam(id, name, city, abbreviation,
                conference, division);
    }
```

Notice, for example, the hard-coded values for `"id"`, `"abbreviation"`, `"city"`, `"name"`. This is based on the output JSON from the balldontlie.io API. You can see the [raw output of the teams page for the API here](https://www.balldontlie.io/api/v1/teams). Very specifically, if I used a message-chain (which is bad code style, but worth showing here) to get this value from the base of the APIs json, I would get a team's abbreviation with:

```java
    JSONObject teamsJSONObject = getTeamsFromBallDontLie();
    String firstTeamAbbreviation = teamsJSONObject.
        getJSONArray("data").
        getJSONObject(0).
        getString("abbreviation");
```

That is, getting the abbreviation is dependent upon 3 nested structures. If the structure or the attribute tags (such as "data") change, this code breaks. Additionally, the API could be offline for a period of time due to no fault of my own, but it would break my app. Or, if I'm using an api-key, that api-key may hit a limit, or even get rejected, due to rules that I have no say in. Even if it's not rejected, I could be paying for the use of a service, and that cost could change suddenly, as has happened with Google Maps who dramatically increased prices in 2018, prompting many developers to switch to tools like OpenStreetMap.

When considering boundaries, you should always have a setup that allows for replacement. A common practice is to hide interactions with external resources behind an abstraction (such as a Java `interface` or `abstract class`). Behind the abstraction is where you implement the details of actually parsing and interpreting data you receive.

This practice is also beneficial if you are preparing for an announced change, or waiting on a release of a new service. You can develop for whatever new data source/format you are preparing for independent of your utilization of your existing data source.

This can also be applied when waiting on a third party program/service to be available. You can develop your own abstract interface for the service that reveals the features you need even before you have access to the technology. Then, once the technology is released, you write an `Adaptor` (which is a Design Pattern we will talk about in the Patterns unit) that translates the information from your external service to the interface your program is already prepared to handle.

### Separation of Concerns

Separation of Concerns is related to modularity. However, when discussing modularity previously, we were focused on individual modules. While architectural components are broader and typically quite a bit more complicated than modules, it doesn't change the fact that we generally want one component to handle one major concern.

For instance, if we have need for persistent data (that is, data that is stored long term, and not reliant on a program actively running in memory of a computer or server), we likely want to create a database. It would be a good idea to separate the connection, maintenance and monitoring of the database as one component.

Additionally, if we're building a web application, we likely will want some form of GUI. This GUI will likely be sent to the user as some HTML file which their local browser will render. In addition to generating these HTML pages and sending them over the internet, you have to consider the likely web browsers your users will have, what features they do or do not support etc. As such, it makes sense to separate the presentation of your website as a separate component.

At core here is often what we mean by "front-end" vs. "back-end" developer. Just like many professions, developers tend to specialize in specific skill sets. The skills and tools used for designing, building, monitoring, and maintaining a database are different from the skills and tools used for building an engaging and effective user interface.

### Long Term Maintainability

When discussing design principles, the main focus was on creating designs that were easy to understand while also being easy to maintain. A good architecture works the same way: an architecture's organization should be clear and understandable. Each component should be cohesive to its role in the larger system, handling one set of interrelated concerns. We want as much functional independence between components as possible. Boundaries between components, therefore should be abstract to hide complexities and implementation details of components from other components.

Just like our modules, if we do not make strong design decisions in our architecture, and practice the discipline to enforce them, software entropy can build up and make our system harder and harder to maintain. With architecture especially, because the decisions regarding how components are built and how they interact often relates to physical systems and large-scale program organization, **architectural designs are the hardest to change.**

## Going forward

In our Patterns unit, we will discuss Architecture again, looking at architectural patterns like:

- Client-Server
- Monolithic
- Component-Based
- Software-as-a-Service
- Microservices
- Layered Architecture
- MVC (we will specifically look at MVC in the context of JavaFX applications)