---
layout: post
title:      "Film Budget Helper - Rails Project"
date:       2018-08-25 20:41:01 +0000
permalink:  film_budget_helper_-_rails_project
---

## Project

The Film Budget Helper is a Rails app designed to help producers track and organize eligible expenses for applications to the New York State Empire State Development Film Production Tax Credit (the tax credit). For more information about the tax credit, visit [their website](https://esd.ny.gov/industries/tv-and-film).

Tracking hundreds of expenses, reimbursing cast and crew members for purchases made with their personal funds, updating cash on hand in real time, and organizing expenses into reports can be tedious and time-consuming. This app is designed to streamline the process for producers and crew members by providing an efficient tool to process reimbursements and organize expenses.

The tax credit requires the following information to qualify an expense:
* Vendor
* Date of purchase
* Department related to the purchase (e.g. craft services, camera, wardrobe)
* Purchase total
* Receipt image
* Location code (the region where the purchase was used)

This is a proof of concept. Additional features still under development include running reports, generating independent budget fields, and reimbursing users.

The main function of this application is to create expenses related to a production with the information required by the tax credit and allow an administrator to approve those expenses.

There are two tiers of users in the app — regular users and administrators. Regular user accounts align with the responsibilities of a crew member on a production. Administrator accounts align with the responsibilities of a producer. Users may create an account using the site’s authentication system or through a Google account. Users can identify themselves as administrators only if they sign up using the site’s authentication system.

On a regular user’s homepage, the user will see a table of the expenses he or she has created with information about the vendor, date, department, total and status. When an expense is submitted, its status is set to pending. This allows an administrator to review the expense and change the status to approved or not approved. If the status is approved, the expense may no longer be edited. If the status is marked as not approved, the user must edit the expense to meet approval guidelines. The regular user's homepage contains links to view the details of the expense, create a new expense, and log out. 

If the user clicks the link to create a new expense, a form to create an expense will be rendered. In this version of the application, vendor, date, department, total, production, and receipt image are all required. There is also an optional field for descriptions. The departments listed come from the tax credit’s guidelines and are set using seed data. Regular users may view and edit their own expenses.

Productions are built by administrators and may not be deleted. A user becomes an administrator by checking the Admin box upon registration. An administrator’s homepage displays a table of all productions and a page to view the individual expenses related to a production which serves as a general ledger. The administrator also has his or her own table of personally submitted expenses at the bottom of his or her home page.

Administrators may edit or delete any expense except those with an approved status. Once approved, an expense will appear on the show page of the production it is related to. Administrators may also view all pending, approved, and not approved expenses using links present on the homepage. Statuses will be connected to reimbursement in future versions of this program.

## Process

This project was difficult to start — I felt daunted by the material and unsure of what I knew. As I dove in deeper, I got hooked on building. Eventually, I needed to rein in my excitement to build new features, and move on. I reminded myself that there’s more waiting for me in JS, React, and Redux! Working on this project built my confidence in refactoring logic into partials, helpers, and model methods. Working with the Rails, Pundit, and ActiveStorage documentation (with the guidance of my Technical Coach Lead) opened my eyes to the process of learning to use programming tools based on web documentation and blogs.
