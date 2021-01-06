---
layout: post
title:      "How to create a calendar todo app with JavaScript and Rails"
date:       2021-01-06 08:38:58 +0000
permalink:  how_to_create_a_calendar_todo_app_with_javascript_and_rails
---


### Introduction
A Rails backend combined with a JavaScript frontend is an excellent way to build out a single-page app. For this project, I decided to create an app that provided a modified calendar for users to view their tasks and daily lists.

The motivation behind this format was to create a more horizontal calendar, one that showed days and tasks from left to right, not just for one week or one month. Tasks belong to initiatives, which help to organize the types of tasks shown on the screen. With my app, anyone can add tasks to a specific calendar day, and see how these tasks stack up across any set of days, given that the viewing window scrolls horizontally. 

### Steps
The way that I separated the project out was as follows:

1. Create database schema
2. Create associated models/controllers for each table in the database
3. Create calendar elements 
4. Fetch data and DOM manipulation

### Database Schema
Although my database schema was simple, it is important to make sure the data for the tasks and initiatives was correct initially:
```
create_table "initiatives", force: :cascade do |t|
    t.string "name"
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
  end

  create_table "tasks", force: :cascade do |t|
    t.string "name"
    t.string "description"
    t.date "start_date"
    t.integer "hours"
    t.boolean "completed_status", default: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.bigint "initiative_id", null: false
    t.index ["initiative_id"], name: "index_tasks_on_initiative_id"
  end
```

Most importantly, foreign keys needed to be added to tasks and initiatives to create the association between the two:
`  add_foreign_key "tasks", "initiatives"`

### Create the Models
Next, the model files were also simple, but included the necessary attributes and validations

```
class Task < ApplicationRecord
  belongs_to :initiative

  validates :name, presence: true
end

class Initiative < ApplicationRecord
  has_many :tasks, dependent: :destroy
end

```

For the Tasks, I also created an associated controller and serializer, to maange how tasks were fetched from the database or created/added to the database:

```
class Api::V1::TasksController < ApplicationController
  def index
    tasks = Task.all
    render json: TaskSerializer.new(tasks)
  end

  def create
    task = Task.new(task_params)
    #byebug
    if task.save
      render json: TaskSerializer.new(task), status: :accepted
    else 
      render json: {errors: task.errors.full_messages}, status: :unprocessible_entity
    end
  end


end

class TaskSerializer
  include FastJsonapi::ObjectSerializer
  attributes :name, :description, :start_date, :hours, :completed_status, :initiative
end

```

### Create Calendar Elements
This reqiured quite a lot of customization, and I almost opted for a preset calendar layout (before making modifications). Ultimately I made my own function `renderCalendar()` that added a `div` element for every day of the 2021 calendar, with month and day labels. Each div element had an id that lined up with its calendar date, in the format of `MM-DD-YYYY`. 

### Fetch data and DOM manipulation
Finally, I needed to use a get request to the database, and find the `start_date` value for each task - this value was then matched with the id of the calendar date div element, and append it to the page.

```
function getTasks() {
  fetch(tasksUrl)
    .then(response => response.json())
    .then(tasks => {
      // remember our JSON data is a bit nested due to our serializer
      tasks.data.forEach(task => {
        let dayContainer = document.getElementById(task.attributes.start_date)
        let newTask = new Task(task, task.attributes)
        const taskMarkup = `
          <div data-id=${task.id} class="box">
            <h3>${task.attributes.name}</h3>
            <p>${task.attributes.description}</p>
            <p>${task.attributes.hours}</p>
            <p>${task.attributes.completed_status}</p>
            <p>${task.attributes.initiative.name}</p>
            <button data-id=${task.id}>Edit</button>
          </div>
          <br><br>`;
        //debugger
        
          dayContainer.innerHTML += taskMarkup
      });
    })
    .catch(err => console.log(err));
}
```

### Troubleshooting
The most challenging aspect was making the id of the calendar date div element precise, so that existing tasks could be displayed, and new tasks could be added to the date - always make sure to `console.log` the id value and the value being stored for the new task to make sure both line up correctly. 

### Conclusion
Thanks for reading, and hopefully you can both use this calendar app and use this article as a model for approaching creating a single-page app. Thanks for reading!



