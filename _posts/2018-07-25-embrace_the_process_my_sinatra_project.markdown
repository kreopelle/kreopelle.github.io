---
layout: post
title:      "Embrace the Process – My Sinatra Project"
date:       2018-07-26 00:53:19 +0000
permalink:  embrace_the_process_my_sinatra_project
---

### The Project

My Sinatra project grew from two pieces of advice:
1. From my TCL — think of it as a glorified spreadsheet
2. From my mother-in-law — make something you would want to use

My previous job was as a film producer. My main project over the past few years was a micro-budget feature film–my first feature and my first fiction film. 

During production, the crew scrambled to catch up at every moment, and eventually we just had to let some things go. One of the things that got postponed until after production was the general ledger — a record of every expense from the production. After we wrapped, I spent weeks manually entering 1000+ expenses into a format required by a state-run tax credit.

As I bounced between paper receipts, company transaction histories, and numerous spreadsheets, I wished there was an easier way to integrate expenses from the production into the proper format and a way to easily track the expenses in real time.

Cue this project: I designed the [Sinatra-Expense-Tracker](https://github.com/kreopelle/sinatra_expense_tracker) as a tool to record expenses for reimbursement.

This is just the first step for a larger application I hope to build during my Rails project that will automate reimbursements, create a general ledger, offer real-time budget data for each department, and output the spreadsheets required for film production tax credits.


## My Process

While building this project, I tried to follow the steps laid out during the Pirates lab walkthrough video: 

1. Build DB
2. Build Models
3. Alternate between creating controllers and views for each model in this order: 
* index
* create
* show
* edit
* destroy. 
Begin with the has_many object controllers and then work on the belongs_to object controllers.

### Some takeaways:
* Sketch out your MVC and object relationships ahead of time. That roadmap will guide you through the build.

* Make sure the foundation of your project (structure, gems, environment, requirements) is laid out before you begin. It’s no fun to chase an error that could have been resolved before you started!

* Follow good CRUD principles and create RESTful Routes. These two concepts take the guesswork out of design and make the project easier to debug while also checking off the boxes in the guidelines

* Work methodically and test consistently. Anytime I tried to jump ahead to the next route or forgot to test something after changing it, I ended up with at least an extra 20 minutes of debugging down the line.

* Know when something works, but don’t be afraid to break it to make your program as a whole function better. This was something I learned in my group Fwitter project that served me well during the Sinatra project. Your methods may depend on each other to work, but that can also blind you to how their relationship might be breaking something else. Question everything and you’ll find the right answer.

