---
title: "Portfolio: RPI Smart Calendar"
date: 2022-12-31 02:45:31
---

<div class="markdown-body">

# Link to the project
- URI: https://rpi-smart-calendar.sycnb.work/
- *Use any email address to receive the Cloudflare OTP*
- Username: test
- Password: CGacZjUZLW9wTud

## Features
#### User Control
- User could login, logout, and register freely

#### Add/Delete Event
- User could add/delete event freely 

#### Study Group
- Students taken in the same courses are assigned to a study group
- They could communicate about the course inside the study group

#### Data Visualization
- Charts indicating average time students spent each week are avaiable
- Students could also see avergae time  they spent on each course

#### Assignment Time Prediction
- Time on future assignments is predicted using machine learning model
- Students could then schedule their time based upon predicted time

## Frontend
- Design pattern: `Composite`
- Technologies
    - `React.js` - JavaScript library for building reactive user interface
    - `Ajax` - Asynchronous JavaScript and XML

## Backend
- Design pattern: `MVC`
- Technologies
    - `Django` - Python-based comprehensive FOSS web framework
    - `Postgres` - Relational database system

## Machine Learning
- Feature Selection
    - Select variables: DueDate, Group, and User, ActualTime
    - Normalized data

- Algorithm: logistic regression with LASSO

## Deployment
![RPI Smart Calendar Deployment Diagram](/images/Time-Master-Deployment-Diagram.png)
- Technologies
    - `docker`
    - `docker-compose`
- Features
    - Easy installation
    - Unify all the config in one file
    - persist data storage
    - system guard and auto restart


# Retrospection
This is a group project. I did most of the work related to coding and documenting, including the overall architecture, the deployment part, the html and javascript parts of the frontend, the majority of the backend, the schema of the database, and the API design for communicating between frontend and backend. To sharpen my skills in React and Django, I did not use any template code and wrote all the codes from scratch.

In retrospect of the project, there are several points that can be improved:
### State Management
To synchronize the changes in different components in the frontend, I had to elevate and aggregate the statuses to parent components. This leads to complex coupling between various components, and the difficulty for maintaince. To address this issue, I may use status management frameworks such as `Redux.js` in future projects.

### Communication Protocal
I utilized the javascript build-in `XMLHttpRequest` for data transferring between frontend and backend. However, I later realized that our implementation is hard to maintain. For future projects, I may adopt some API framework such as `GraphQL` that is understandable, predictable, and maintainable.

### Intuitive, Unified, and Elegant UI Design
We didn't develop a solid consensus for the UI design. Since everyone participates in some parts of the CSS, our CSS reflects our different aesthetic senses. For future projects, we can document the UI design and create a demo UI before coding. We may also use some element-wise templates such as `Bootstrap`, `BULMA`, and `Animate.css`.
</div>