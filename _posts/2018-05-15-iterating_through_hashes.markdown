---
layout: post
title:      "Iterating through hashes"
date:       2018-05-15 22:00:08 -0400
permalink:  iterating_through_hashes
---


As a baker by day and programming student by night, I find a lot of connections between my two worlds. Variables remind me of ingredients (type_of_flour = bread flour). Instructions remind me of loops (knead dough 10 times). Knowing when and how to proceed reminds me of conditional statements (if the dough is sticky, add flour, else if the dough is dry, add water, else begin rise).

One of my earliest lessons with baking bread was to measure my ingredients as precisely as possible. When baking a loaf or two at home, an extra tablespoon of flour can easily be compensated for, but when baking 200+ loaves before the Easter rush, you better get it right. There are enough other factors (mixing, proofing, shaping, etc.) that require attention and discretion that I found beginning with a groundedness in my ingredients being measured properly made a huge difference. 

![With all of these potential issues, why worry about measuring if you don't have to? Photo from Serious Eats](https://www.seriouseats.com/images/2014/11/bread-infographic2-forweb.jpg)

>[With all these potential issues, why worry about measuring if you don't have to? Photo from Serious Eats by Vicki Wasik.](https://www.seriouseats.com/2014/11/troubleshoot-bad-bread-messed-up-loaf.html)

Unfortunately, the United States is stubborn about their independent units of measurement and still leans more toward cups than the precision and universality of grams. Cups can be tricky – they're imprecise. *My* one cup may not hold quite the same volume as *your* one cup. With the potential for flour to pack down into the cup, measuring ingredients on a scale helps eliminate some of the guesswork (and you can just dump everything into a bowl to check the amount). 

![Sarah Jampel of Food52 found that there could be as much as a 37-gram difference in the volume of sugar in a measuring cup based on which cup you use.](https://images.food52.com/bVlnUPAnkvV_c97GkJiViXD1Vxc=/fit-in/800x0/f4605c1a-be2e-4671-b687-b99cb589311a--IMG_6657.JPG)

> [Sarah Jampel of Food52 found that there could be as much as a 37-gram difference in the volume of sugar in a quarter-cup measure based on your set. Photo from Food52.](https://food52.com/blog/16497-the-truth-about-your-measuring-cup-dun-dun-dun)

Unwilling to kick my favorite American recipes to the curb, but believing in an easier way than researching, converting, and writing each ingredient down when I want to use my kitchen scale instead of measuring cups -- why not create a method to do the job for me?! 

For this experiment, I decided to use [Bobby Flay's Pizza Dough](https://www.foodnetwork.com/recipes/bobby-flay/pizza-dough-recipe-1921714), my go-to for a quick, foolproof dinner.

I decided that the best way to store the ingredients was in a hash. Hashes are dictionary-like collections of key-value pairs. In terms of a recipe, keys could be thought of as ingredients and values could be thought of as the quantity of the ingredient to use in the recipe. 

I transformed this: 

**Ingredients**
* 3 1/2 to 4 cups bread flour, plus more for rolling (Chef's Note: Using bread flour will give you a much crisper crust. If you can't find bread flour, you can substitute it with all-purpose flour which will give you a chewier crust.)
* 1 teaspoon sugar
* 1 envelope instant dry yeast
* 2 teaspoons kosher salt
* 1 1/2 cups water, 110 degrees F
* 2 tablespoons olive oil, plus 2 teaspoons


... into this: 

```
pizza_dough_ingredients = {
  bread_flour: "3.5 cups",
    sugar: "1 teaspoon",
    instant_dry_yeast: "1 envelope",
    kosher_salt: "2 teaspoons",
    water: "1.5 cups",
  olive_oil: "2 tablespoons"  
}
```

Originally I tested my method using a list of variables, with each ingredient set equal to its quantity, however, it made it difficult to see the two together after running them through a method. I wanted the two values to stick together as I converted the quantity from cups to grams, so key-value pairs seemed more reasonable.

Next, I worked on my method. Fortunately, the values of the ingredients all shared a similar pattern that I could use to my advantage. They started with a number and were followed by the unit of measurement. I wrote out some pseudocode to map out what I wanted to accomplish: 

```
# go through the entire collection of ingredients 
# separate the amount from the unit of measurement
# perform math to change the value to its equivalent in grams
# put all data back together into a readable list 
```

Then I tested my process with just a single ingredient. I wanted to make sure that the code I'd be using to iterate over the hash did what I wanted it to do before running it over the entire list. According to [Convert Units](https://www.convertunits.com/from/teaspoons/to/grams), one teaspoon should be equal to five grams. 

```

sugar = "1 teaspoon"

def gram_converter(ingredient)
  ingredient_array = ingredient.split 

# separate the quantity from the unit and save them into a new variable. #split returns an array.

  if ingredient_array[1] == "teaspoon"
    
# the second value of the array should be the unit of measurement. 
# this checks to determine if it is a teaspoon

    ingredient = ingredient_array[0].to_i * 5
    
# the first value in the ingredient array should be the quantity.
# if the value is a teaspoon, then convert it to an integer and multiply it by 5 
# if it is not converted to an integer, the quantity will just appear five times

    puts ingredient

# reveals the new value of the ingredient 

  else 
    puts "error"
        
# if it didn't work, lets me know so that I can go back to check my work

  end
  
end

gram_converter(sugar)

# => 5

```

Once I got this part of the method working, I moved over to the hash iteration. Hash iteration works like iterating over any other form of data, except it accepts two parameters––the key and the value. For this method, I set the key equal to an ingredient, and the value equal to its quantity. 

```

def gram_converter(ingredient_hash)
  ingredient_hash.each do |ingredient, quantity|
    # code to perform on each key-value pair
    end 
end

```

Now, I needed to account for the different units of measurement commonly found in American recipes -- teaspoons, tablespoons, and cups. I created a series of conditional statements that would operate on the quantity if a given unit word was included in the string: 

```
def gram_converter(ingredient_hash)

  ingredient_hash.each do |ingredient, quantity|
    
    quantity_array = quantity.split 
        
    if quantity_array[1].include?("teaspoon")
      quantity = quantity_array[0].to_i * 5
            
    elsif quantity_array[1].include?("cup")
      quantity = quantity_array[0].to_i * 236.58
            
    elsif quantity_array[1].include?("tablespoon")
      quantity = quantity_array[0].to_i * 15
            
    else 
      puts "error, unable to convert '#{ingredient}: #{quantity}'"
   
     end
  end
end
```

The format was similar to my test code: split the value and save it in a new variable, check to see which unit of measurement is being used for that ingredient, transform the quantity into an integer and multiply it by the proper adjustment in grams. By setting the #include? method to "cup", it allowed both the writing of "cup" and "cups" to be evaluated for the operation.

Though this solution worked well in some respects, it had a few huge errors:
* It didn't account for the possibility that some ingredients may not be integers, but floats. With flour, for example, the program evaluated 3 cups of flour instead of 3.5, resulting in 709 grams of flour instead of the accurate 828 grams. 
* The return value did not print the adjusted ingredients, only the error message and the original hash
* The program didn't adjust the value of instant_dry_yeast

To fix the bugs: 
* I converted ```quantity_array[0]``` to a float instead of an integer, and then later returned it to the more pleasing-to-the-eye integer value.
* I didn't want to lose the original recipe's values, so instead I added a puts line that printed the ingredient and its quantity in a more readable way
* I looked up and added a line for "envelope" to convert the value of instant_dry_yeast

Now, my code read:

```
def gram_converter(ingredient_hash)

  puts "Ingredients converted to grams"

ingredient_hash.each do |ingredient, quantity|

quantity_array = quantity.split 
    
        if quantity_array[1].include?("teaspoon")
      quantity = quantity_array[0].to_f * 5
      quantity = quantity.to_i 
      puts "#{ingredient}: #{quantity} grams"
    
        elsif quantity_array[1].include?("cup")
      quantity = quantity_array[0].to_f * 236.58
      quantity = quantity.to_i 
      puts "#{ingredient}: #{quantity} grams"
    
        elsif quantity_array[1].include?("tablespoon")
      quantity = quantity_array[0].to_f * 15
      quantity = quantity.to_i 
      puts "#{ingredient}: #{quantity} grams"
    
        elsif quantity_array[1].include?("envelope")
      quantity = quantity_array[0].to_f * 7
      quantity = quantity.to_i 
      puts "#{ingredient}: #{quantity} grams"
    
        else 
      puts "error, unable to convert '#{ingredient}: #{quantity}'"
    
        end
  end
end
```

I couldn't help but feel like there was far too much repetition in this code and remembered that I had learned about a way to re-write conditional statements to perform relatively similar operations on code with different values. In came the case statement! 

I didn't remember the format well, so I hopped on Google and found [a great article on Skorks](https://www.skorks.com/2009/08/how-a-ruby-case-statement-works-and-what-you-can-do-with-it/) that walked me through it. Now, my code performs the same task, but looks a lot cleaner: 

```
def gram_converter(ingredient_hash)

  puts "INGREDIENTS CONVERTED TO GRAMS"
    
  ingredient_hash.each do |ingredient, quantity|
    
    quantity_array = quantity.split
   
     case quantity_array[1]
        
    when "teaspoon", "teaspoons" 
      quantity = quantity_array[0].to_f * 5
   
     when "cup", "cups"
      quantity = quantity_array[0].to_f * 236.58
    
        when "tablespoon", "tablespoons"
      quantity = quantity_array[0].to_f * 15
    
        when "envelope", "envelopes", "packet", "packets"
      quantity = quantity_array[0].to_f * 7
   
     else 
      puts "error, unable to convert '#{ingredient}: #{quantity}'"
    end
    
        puts "#{ingredient}: #{quantity.to_i}"
        end
  
    puts "ORIGINAL: #{ingredient_hash}"
end

``` 

So when I call ```gram_converter(pizza_dough_ingredients)``` I get:

```
INGREDIENTS CONVERTED TO GRAMS
sugar: 5
bread_flour: 828
olive_oil: 30
instant_dry_yeast: 7
water: 354
ORIGINAL: {:sugar=>"1 teaspoon", :bread_flour=>"3.5 cups", :olive_oil=>"2 tablespoons", :instant_dry_yeast=>"1 envelope", :water=>"1.5 cups"}
```

This method just scrapes the surface of useful ingredient conversion. Could I re-write it to pull the conversions from a database to include even more measurement types (such as the information for the grams in an envelope of yeast instead of adding it as another elsif statement)? How could I account for ingredients that are used multiple times (such as olive oil in the original recipe first being used in the quantity of two tablespoons and later on being used in the quantity of two teaspoons)? Is there a way to preserve ranges of quantities, such as 3.5 to 4 cups of bread flour, and have both of the values converted? How might I adjust the program to simply copy and paste the ingredient list into a user input field, and have the program itself format it into a hash before converting the data to grams?

As I learn more about programming, I hope to come back and update this method to incorporate the functionality mentioned above. For now, I'm just thrilled to have a way to easily get a list of my ingredients without spending too much time doing the math. 

*Do you have any ideas to make my code terser? Do you have any answers to the questions above? What are some of your favorite applications of programming in your daily life?*
