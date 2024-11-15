+++
date = '2024-11-14T20:01:13-05:00'
draft = false
title = 'Day #7: Increasing the Difficulty Level'
tags = ['#100DaysOfCoding']
+++
For no reason whatsoever, I decided that learning Python wasn't difficult enough. So, I committed to writing all of [today's lesson code in Neovim](https://neovim.io/). I thought it would be fun to leave the fancy VS Code behind with all the auto-complete, linting, and mouse input and try to produce code in Neovim.

>Just me, my code and a bunch of weird keyboard commands to move the cursor around my document.

It will come as no surprise this choice slowed down my progress today, but I don't think it's a bad thing. I love the simplicity of the Vim interface. Maximize the terminal window, and it's like a Zen mode. No IDE. No menus. No mouse. No distractions. Just me, my code and a bunch of weird keyboard commands to move the cursor around my document.

![](/images/day-7-neovim.png)

## Making Simple Things More Complicated

I continued [to work through the Bro Code Python course](). I'm 3 hours and 19 minutes into the 12-hour course. We've moved into lists, sets, tuples and dictionaries. Still fairly basic, but adding in the extra challenge of coding in Terminal with Neovim ensured the difficulty was sufficient. 

>...but adding in the extra challenge of coding in Terminal with Vim, the difficulty was sufficient. 

The last exercise I copy-coded was creating a concession stand app to calculate the order total. I didn't pick up anything new in Python, but by the end of the lesson, my Vim skills were slightly more proficient. 

```
menu = {"Pizza": 3.00, "Nachos": 4.50, "Popcorn": 6.50, "Fries": 2.50, "Chips": 1.50, "Pretzel": 3.50, "Soda": 3.00, "Lemonade": 4.50}

cart = []
total = 0

print("----- MENU ------")
for key, value in menu.items():
    print(f"{key:10}: ${value:.2f}")
print("-----------------")

while True:
    food = input("Select an item: (Q to Quit) ")
    if food.lower() == "q":
        break
    elif menu.get(food) is not None:
        cart.append(food)

print("------ YOUR ORDER ------")

for item in cart:
    total += menu.get(item)
    print(item, end=" ")

print()
print(f"Your total is: ${total:.2f}")
```
Neovim also supports Markdown, so I pushed my luck and started typing today's blog post in the terminal as well. That didn't last long. Maybe one day I'll get there, but for now, I'm [sticking with MacDown](https://macdown.uranusjr.com/) for the blogs.

## Tackling a 1,000 Digit Number

While I had Neovim open, I thought I would take another shot at [Project Euler #8](https://projecteuler.net/problem=8). The challenge is to find out which 13 adjacent digits multiplied together produced the highest number.

As I'm still well below Project Euler's limit of 100 for sharing solutions outside their platform, I don't mind sharing the script I came up with to find the magic number.


```
series = "7316717653133062491922511967442657474235534919493496983520312774506326239578318016984801869478851843858615607891129494954595017379583319528532088055111254069874715852386305071569329096329522744304355766896648950445244523161731856403098711121722383113622298934233803081353362766142828064444866452387493035890729629049156044077239071381051585930796086670172427121883998797908792274921901699720888093776657273330010533678812202354218097512545405947522435258490771167055601360483958644670632441572215539753697817977846174064955149290862569321978468622482839722413756570560574902614079729686524145351004748216637048440319989000889524345065854122758866688116427171479924442928230863465674813919123162824586178664583591245665294765456828489128831426076900422421902267105562632111110937054421750694165896040807198403850962455444362981230987879927244284909188845801561660979191338754992005240636899125607176060588611646710940507754100225698315520005593572972571636269561882670428252483600823257530420752963450"

largest_product = 0

for i in range(len(series)):
    product = 1
    adjacent_digits = series[i:i+13]
    for j in adjacent_digits:
        product *= int(j)
    if product > largest_product:
        largest_product = product
    
print(f'Largest Product: {largest_product}')
```
I started a loop on the first digit and iterated over the next 12 to find the product. If the product was higher than the previous iteration, I updated my ```largest_product``` variable with the new product. At the end of the loop (1000 iterations later), I had the total. Quickly copy and paste into the Project Euler website, and it's time to write a blog post.

## Brilliant is Brilliant

But before I do that, I want to plug the brilliant Brilliant app. I'm enjoying it very much. I downloaded it a few days ago and did the first ten lessons in the Computer Science course. Simple programming challenges with block code. I love this style of app, [as I've mentioned before](/posts/day-1-laying-the-groundwork/). The game element makes the repetition more enjoyable. 

Deciding I wanted the full version of the app, I went back to one of my [favourite chess YouTube channels, ASMRchess](https://www.youtube.com/@ASMRChess). He's advertised Brilliant in several of his recent videos. Using his product code, I was able to get a 30-day free trial (beats the dozen lessons you bet with the base app) and a discount off the annual subscription rate.

I'll be sure to get plenty of value out of my free trial, but I've already made the decision to purchase the subscription when the time comes. Thirty days is great...

But we still have 93 more days to go...




