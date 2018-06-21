---
layout: post
title:      "Building the CLI Data Gem Project"
date:       2018-06-21 19:47:44 +0000
permalink:  building_the_cli_data_gem_project
---


### Overview

For my CLI Data Gem Project, I wanted to build an app that would utilize data I wanted to learn more about. I've practiced yoga for years, and when I came across [Yoga Journal’s pose library](https://www.yogajournal.com/poses) and ran some preliminary scrapes in pry, I knew I found my idea. 

### Development

Originally I wanted to use the entire [Poses by Type](https://www.yogajournal.com/poses/types) page to print a list of each pose type, print all of the poses within that type, and then allow a user to receive detailed information about a selected pose. After I discovered the project required scraping only two levels of data, I decided to focus on [Strengthening Yoga Poses](https://www.yogajournal.com/poses/types/strength) and consider building an extension to include all pose types at another time. 

I took to heart that the aim of the project to write good object-oriented ruby code and not show off scraping prowess. I wanted to knock out the scraping selectors before starting on the architecture to let the program's design take center stage. 

Grabbing the 'Sanskrit Name' and 'Beginner's Tip' stumped me. I decided to table finding the selectors for those two attributes and focus on getting the rest of the app up and running. 

I was grateful for the support of the CLI Data Gem Study Group session to get my project up and running. Using Bundler to create a gem made sense during the README, but I didn’t really understand the process until I set it up for my app. Advice to change the repository's name to snake case made a huge difference — storing my app in a single directory instead of three nested directories. I followed Kenlyn’s advice to [obey the seven rules of a great Git commit message]( https://chris.beams.io/posts/git-commit/) and eventually got into a flow of adding, committing and pushing my changes to GitHub.

I followed Avi's process as demonstrated in the CLI Data Gem Walkthrough. I created a working version of the user interface with hard-coded data, transitioned to developing a hard-coded version of Asana objects before scraping the real data. It took a while to get used to the patterns and standard placements for `require` and `require_relative` statements. 

When it was time to return to find the selectors for ‘Sanskrit Name’ and ‘Beginner’s Tip’, I spent a few hours trying to figure it out on my own. Realizing I ran out of ideas and was wasting time, I asked my peers for support. We worked together to determine a selector that could grab both pieces of data. I learned about CSS combinators and utilized the adjacent sibling combinator. My favorite Eureka moment of the project came after struggling to create an iterator that would access the specific `<p>` tag I was looking for based on the text of the `<h3>` tag above it. I tried conditional statements and #detect to no avail. Finally, I realized I could store `<h3>`  and their adjacent `<p>` siblings as key/value pairs in a hash!  Iterating through the `<h3>` tags in this way allows for the extension of the program in future versions to give the user access to other pieces of data. The only problem with my solution was that it grabbed only `<p>` tags, which worked for my purposes, but later I would also like it to grab the adjacent sibling `<ul>` tags in the same div.

I must admit, after the first successful run of the app I was so excited that I ran upstairs, shook my sleeping spouse awake, and bounced around the room, babbling about my good news.

I made a deal with myself to hold off on watching the anti-pattern and refactoring videos until I had a functioning project. I felt pretty confident about my code but was encouraged by the walkthroughs to reorganize my Scraper class to decrease the load time of the project. I set aside my pride, commented out my code, and tinkered until I designed new methods that first scraped just the Strengthening Yoga Poses page for the list view, and then scraped the individual yoga pose pages upon user input. It felt scary, I knew there was a chance I wouldn’t be able to figure it out, and would be stuck with code that functioned, but didn’t have that Ruby elegance. Successfully refactoring those methods brought another burst of joy, proving to myself, yet again, I underestimate my own abilities. 

### What I got out of this project 

This project gave me a new confidence in web development. I can see how this simple CLI app lays a foundation for much more complex projects down the road. Applying patterns and concepts from the curriculum felt like having my first conversation in a new language. I recognized just how much support the LearnIDE and lab setup provides to create a space to practice concepts, as opposed to the additional distractions of creating and maintaining your own repository. Presenting creative work requires vulnerability. I have a bad habit of waiting I feel absolutely certain something is without critique before sharing. Web development is teaching me that not only do I need to become comfortable with failure, I need to embrace it. If I don’t try, the code won’t get written, the bug won’t be fixed, and the functionality won’t be improved. There’s a balance between accessing my own knowledge and humbly asking for support. I recognize that I am still very much a beginner, but feel fortunate to be supported by a community of learners who strive to succeed together. 

