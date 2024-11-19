+++
date = '2024-11-10T02:53:00-05:00'
draft = false
title = 'Day #3: The Value of Coding Challenges'
tags = ['#100DaysOfCoding']
+++
In yesterday's post, I attempted to illustrate the [differences between 'copy-coding' tutorials and actually writing code](day-2-coding-tutorials-vs-actually-coding/). Even after just a few days of my project, I've already noticed the code I remember is the code I wrote without following a tutorial.

The problem is I'm never sure what to write. I don't have a project in mind for my Python code, so I've been looking for ideas. This is where I discovered the concept of coding challenges. I started with a course from [Code with Mosh](https://codewithmosh.com/) that contains a dozen or so Python projects in two-part videos. The first one lays out the challenge. The second shows his solution. I found these interesting. 

The trick was to watch the first video and then complete the project on my own, including the extra features listed in the accompanying PDF. Once my program was functional, I would watch the second video and see where I could improve.

## Completing Coding Challenges

The first project was a dice-rolling game. The premise was to write a script that would prompt the user to roll two dice and then return the values. Fairly straightforward. The extra credit challenge was to allow the user to select the number of dice to roll and count the total number of rolls. Here's what I came up with:

```
import random

print('\nWelcome to the Dice Rolling Game!\n')

counter = 1

while True:
    user_choice = input("Roll the dice? (y/n): ")

    if user_choice.lower() == 'y':
        number_of_dice = input('How many dice would you like to roll? ')
        list_of_results = []

        for i in range(int(number_of_dice)):
            list_of_results.append(random.randint(1, 6))

        print(f'\nRoll {counter}: {list_of_results}\n')
        counter += 1
    elif user_choice.lower() == 'n':
        print('\nExiting game...\n')
        break
    else:
        print('\nYou should make better choices!\n')
 ```
I made it through the first two lessons and am working on the third, a rock-paper-scissors game. You can view the code for [these projects in my GitHub repository](https://github.com/stevesansford/python-projects/tree/main/code-with-mosh).
 
## What About Free Coding Challenges?
 
The disadvantage is this challenge course is not free. Mosh sells his courses, and while they do provide good value, there are similar resources available for free that would serve the same purpose. Although, the solutions may not be as detailed.

One suggestion I have not yet tried is to use ChatGPT (or another similar platform) to provide a series of beginner challenges and a series of tests to prove the solution. I'm doubtful this will work for more complicated programs, I have enough experience with ChatGPT to believe that it can produce challenges suitable for my current skill level. I've added this to my list of coding things to try, so check back in a few days. 
	
One thing of interest I did find was [Project Euler](https://projecteuler.net/about). It's a series of 904 (at last count) challenges, starting with basic and progressing to considerable difficulty. The problems are math-based, and although I have successfully completed the first eight challenges, I found that the limiting factor was my understanding of mathematical concepts and not my coding. 

The Project Euler website is dated but serves its purpose. The advantage is there's no language requirement. You can use any language you want to determine the answer to the problem. FreeCodeCamp.org has the first 480 lessons available on their platform in JavaScript. If that's your language, you may find the interface on FreeCodeCamp.com more useful. As I'm using Python, I wrote my code in VSCode and then manually entered my answer on the Project Euler website. I'm enjoying this challenge very much, and it's helping me to expand both my coding ability and mathematical skills.

## Coding Challenges Produce Results

So far, I've found this type of challenge the most productive. Not only do I need to learn the Python syntax necessary to provide the solution, but I also need to apply logic and problem-solving skills (and let's not forget the math) to figure out what I need to do before I even attempt to write the code. Project Euler suggests starting with a pencil and paper until you have an understanding of the problem and an outline to solve the problem. That's a great suggestion for any programming project.

We've only got 97 days to go...
 
 