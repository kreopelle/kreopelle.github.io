---
layout: post
title:      "Nokogiri Scraping Tutorial"
date:       2018-06-12 17:11:07 +0000
permalink:  nokogiri_scraping_tutorial
---


Before we get started, I want you all to know that I am not an expert in Nokogiri and have just begun my CLI Data Gem Project. However, I am curious about how gem works. This post walks through my process of scraping data. 

[Nokogiri](http://www.nokogiri.org/) is a Ruby gem designed to scrape data from websites. To use it, all you need is a little CSS knowledge, Ruby fundamentals, and patience. 

For this walkthrough, I’m using [Yoga Journal’s Poses by Type](https://www.yogajournal.com/poses/types). 


![Yoga Journal's Poses by Type Webpage](https://imgur.com/FRr2M3A)


This is one of the pages I'm using for my CLI Data Gem Project. Scraping sites can fail if their content or structure has changed, so this content is accurate as of publication on June 12, 2018. I'll also be using the Google Chrome browser on a Mac. Feel free to follow along!

###  1. Add Nokogiri, Open-URI, and Pry to your ruby file 

First, install the gems by typing into the terminal:
```
gem install nokogiri
gem install pry
```

Once installed, require them at the top of your ruby file:
```
require ‘nokogiri’
require ‘open-uri’
require ‘pry’
```

Note: Open-URI is not a gem, but a built-in Ruby wrapper that allows HTML documents to be read as a file: https://ruby-doc.org/stdlib-2.1.0/libdoc/open-uri/rdoc/OpenURI.html. I’m not sure how it relates to the current Ruby series.  

### 2. Save the site you’d like to scrape to a variable

`doc = Nokogiri::HTML(open(‘https://www.yogajournal.com/poses/types'))`

### 3. Add a binding.pry below the variable and run your file. 

If you’re following along, the document should look something like this:

```
require ‘nokogiri’
require ‘open-uri’
require ‘pry’

doc = Nokogiri::HTML(open(‘https://www.yogajournal.com/poses/types'))
binding.pry
```

And your terminal command to run the file would be:

`ruby my-file.rb` 

### 4. Inspect content to scrape on your webpage

Let’s hop over to the website. For part of my project, I'm hoping to create a class of objects where each instance is a type of yoga pose. I’d like to use Nokogiri to scrape all of the pose type names and their associated URLs. 

Right click on the page and hit *Inspect*. Go to the top left corner of the inspect window, and click on the arrow pointing to the middle of the square (the info balloon should say something like, “Select an element in the page to inspect it”). 

![Right click, then select 'Inspect'](https://imgur.com/rS5ViSy)

![Locating the Element Inspector Icon](https://imgur.com/sRm1YT2)

Hover over an example of the item you’re looking for. This will highlight the corresponding CSS in the Elements window while highlighting the element’s bounds (usually through a colored box). Click on the item to select its CSS in the Elements window. 

When hovering, I like to ask myself:

* Is the content I’m looking for stored within this item? Can I see the text, link, image, etc. wrapped within the CSS in the Elements panel? 

![While hovering, look at the highlighted content in the Element Inspector Panel](https://imgur.com/wmeoAwG)

Yes! between the > < signs, I see “Arm Balance Yoga Poses”

* What is the CSS selector or HTML tag for this item? Some common examples are `div`, `a`, and `h2`. In Chrome, this will be the text that shows up first, in pink.

![Close up hovering view of Arm Balance Yoga Poses](https://imgur.com/ZQk4oH3)

For this example, the selector is `h2`.

* Is there any ID or Class information that follows the selector/tag? 

![Close up hovering view of Arm Balance Yoga Poses](https://imgur.com/ZQk4oH3)

Here, the text is a little long. It looks like the class is called `m-card--header-text ng-isolate-scope`. Double click on the class information in the Elements panel and copy it.

*Tip: in Google Chrome, while hovering your cursor over the item you’d like to scrape, the format already matches what you need to put into your Nokogiri scrape argument! If it’s an ID, the text will have a '#' in front of it. If it’s a Class, the text will have a '.' in front of it. In Chrome’s representation, id’s are orange and classes are blue.*

* Does similar content on the page share CSS selectors/classes, etc? Hover over similar items to see if they share selector/class/id information. 

![Close up hovering view of Balancing Yoga Poses](https://imgur.com/HP9VtTF)

In this case, yes: "Balancing Yoga Poses" has the same CSS as "Arm Balances"! 

Let's take this information back to our code.


### 5. Play around in Pry

Experimentation time! 

The standard structure for scraping using CSS in Nokogiri is `variable.css(‘selector#id.class’) `, so in this example, `doc.css(‘h2.m-card--header-text ng-isolate-scope’)` is the first thing I’ll run. 

Hmmm, looks like we’re getting back an empty array. I was a little skeptical about the class because of space in-between 'text' and 'ng'. When using Nokogiri, spaces can indicate moving through different nested levels. I’ve also noticed that successful Nokogiri scrapes are often simple and do not contain the full CSS. Let’s try pairing it down to include only the class name before the space. 

`doc.css(‘h2.m-card--header-text’)`

*Tip: Hit just the up arrow while in the terminal to get the previous typed value.*

Now we’re getting somewhere! In the returned data, I can see the strings I want to pull, but I need to add a bit more specificity to get just the pose type category name.

Hit ‘q’ to exit the returned value and get back to a space to type new commands. 

To specify the content you’re searching for in Nokogiri, chain additional methods after `#.css`. I most frequently use:

```
#.text —> will pull strings 
#.attribute('href').value —> will pull urls 
#.attribute('img').value —> will pull image urls 
```

In this case, we’re looking for a string, so let’s try adding `#.text `

```
pry(main)> doc.css('h2.m-card--header-text').text
=> "Arm Balance Yoga PosesBalancing Yoga PosesBinding Yoga PosesChest-Opening Yoga PosesCore Yoga PosesForward Bend Yoga Po
sesHip-Opening Yoga PosesInversion Yoga PosesPranayama Exercises & PosesRestorative Yoga PosesSeated Yoga PosesStanding Yog
a PosesStrengthening Yoga PosesTwist Yoga PosesYoga BackbendsYoga BandhaYoga Mudras8 Top Yoga Teachers Give Their Best Advi
ce for Finding Bravery in Inversions3 Ways to Modify PaschimottanasanaMaster Paschimottanasana in 6 StepsThe Bandha Approac
h You Haven't Tried—That Could Change EverythingThe Muscle That Can Make or Break Healthy Shoulders in InversionsMaster Cla
ss: How Does Pranayama Help Digestion?This Energy-Boosting Breathwork Is Better Than CaffeineMaster Class: Rodney Yee’s 3-S
tep Pranayama Technique for Stillness and PeaceYoga Journal's 2017 Pose of YearRodney Yee's Restorative Yoga Sequence to Pr
epare for Pranayama"
```

Woohoo! We’ve got one long string that has all pose type headlines! To determine if they are accessible as individuals, I like to run the same code, but chaining `#.first `before `#.text`

```
pry(main)> doc.css('h2.m-card--header-text').first.text
=> "Arm Balance Yoga Poses”
```

Awesome. We’ve got you now. 

### 6. Push the content into an array

Return to your web browser. Take a look at the parent of your desired data. Look for an arrow pointing down in the Elements panel.

In this case, it’s:

![View of the parent element](https://imgur.com/UPbbmYM)

I tend to have a lot of luck with using only the class for the method's argument when building an array of data scraped from a webpage. Let’s test the following: 

`doc.css('.m-card—header')`

Sweet — our little strings are nestled within this code. Let’s try iterating through it: 

```
doc.css('.m-card--header').collect do |pose_type|
    pose_type.css('h2').text
end 
```

Here, I’m using the `doc` variable, saved to the value of the HTML of our desired webpage, and narrow it down to just the data that has a class of `.m-card--header`. I’m using the `#collect` iterator to go through each child of this class–which in our case, we hope is a pose type category–and push just the text within `h2` tags into an array. 

Why just `h2`? Nokogiri doesn’t require both tags/selectors and classes/ids in every call. Sometimes one or the other will retrieve what you’re looking for. In this case, the `.m-card--header` class narrows the retrieved content to children of that class. Each headline is encased within an `h2` tag, and the only `h2` tags that are children of `.m-card--header` contain the data I’d like to scrape. 

In this case, success!! 

```
=> ["Arm Balance Yoga Poses",
 "Balancing Yoga Poses",
 "Binding Yoga Poses",
 "Chest-Opening Yoga Poses",
 "Core Yoga Poses",
 "Forward Bend Yoga Poses",
 "Hip-Opening Yoga Poses",
 "Inversion Yoga Poses",
 "Pranayama Exercises & Poses",
 "Restorative Yoga Poses",
 "Seated Yoga Poses",
 "Standing Yoga Poses",
 "Strengthening Yoga Poses",
 "Twist Yoga Poses",
 "Yoga Backbends",
 ...
```

Don’t be afraid to keep playing around until you find what works, or go back to the CSS to see if there may be another way to scrape what you’re looking for. 

I noticed some of the headlines for articles at the bottom of the page got pulled as well, which means I’ll have some additional drilling/data verification to do when I get to my CLI project to retrieve just the pose type category names.

Links are a little trickier. To get the link each pose type points to, I wasn’t able to hover, I needed to click on the title and look within the element's data. When I hovered over the 'a' tag above the 'h2' tag in the Elements panel, the element that contained the link on the page highlighted. That helped me confirm the link data I wanted to scrape was encapsulated within the highlighted CSS element. 

To return links instead of text, the Nokogiri format is: 

`doc.css('a.class-name).attribute('href').value `

And to iterate through them, use just the ‘a’ tag for the collection, followed by the .attribute(‘href’).value chain as the block. To get the links that accompany these pose types, I used: 

```
doc.css('a.m-card—header').collect do |pose_link|
    pose_link.attribute('href').value
end 

=> ["/poses/types/arm-balances",
 "/poses/types/balancing",
 "/poses/types/binds",
 "/poses/types/chest-openers",
 "/poses/types/core",
 "/poses/types/forward-bends",
 "/poses/types/hip-openers",
 "/poses/types/inversions",
 "/poses/types/pranayama",
 "/poses/types/restorative",
 "/poses/types/seated-twists",
 "/poses/types/standing",
 "/poses/types/strength",
```


And that’s it! I’m learning more Nokogiri while working on the CLI Data Gem project, and hope to add more tips and tricks to add to this post as I better understand its functionality.
