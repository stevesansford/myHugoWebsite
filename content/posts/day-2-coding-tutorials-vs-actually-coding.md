+++
date = '2024-11-09T02:52:35-05:00'
draft = false
title = 'Day #2: Coding Tutorials vs. Actually Coding'
tags = ['#100DaysOfCoding']
+++
The sheer amount of coding material on the Internet is overwhelming. It's easy to get lost in what the community calls 'Tutorial Hell.' Students get stuck in an endless process of watching tutorials on YouTube for hours on end, with no appreciable gain in skill.

I'm not saying that it's impossible to learn coding from watching videos, but it's sort of like learning to ride a bike by reading a book about riding a bike. It can tell you all the theories and best practices, but until you get out on your bike and fall off a few times, you'll never learn how to do it.

Coding is the same. This isn't to say coding tutorials have no value. So far in my short journey, I've found them to provide tremendous value when used in a specific way.

## 'Copy-Coding' as a Discovery Tool

'Copy-coding' as I call it, is the process of following along with a tutorial and copying the code they write line by line until you end up with a completed project.

I get two benefits from this process. First, I learn what to expect from the language. I don't focus on the syntax and structure of the code. Rather, I try to focus on the idea behind the code. What is the programmer trying to accomplish? How does the language enable the solution?

Second, even though my focus isn't on the syntax or structure, I can't help but absorb the patterns. Certain structures start feeling natural even after only a few hours of coding. These are basic structures but essential to a fundamental understanding of the code.

## The Real Trick to Learning a Skill

Learned knowledge is powerful, but it cannot compare to practical knowledge. Just like the best way to learn to ride a bike is to jump on one and give it a shot, the best way to learn coding is to jump in, sans tutorial, and just figure stuff out.

It's not easy to sit down in front of a blank editor and just bang out some code, so I've found some structures I can use to help jumpstart the process.

My favourite so far is the courses on FreeCodeCamp.org. I know, that's painfully obvious. The lessons are well-built and start with a series of instructions that build out a functional application from the ground up. There are elements of 'copy-coding' in some of the steps, but more than a few required me to go look up a solution in the Python docs. That's how to learn.

Here's an example of one of the lessons where I followed the instructions (and looked up a few things) to create a random password generator:

```import re
import secrets
import string


def generate_password(length=16384, nums=1, special_chars=1, uppercase=1, lowercase=1):

    # Define the possible characters for the password
    letters = string.ascii_letters
    digits = string.digits
    symbols = string.punctuation

    # Combine all characters
    all_characters = letters + digits + symbols

    while True:
        password = ''
        # Generate password
        for _ in range(length):
            password += secrets.choice(all_characters)

        constraints = [
            (nums, r'\d'),
            (special_chars, fr'[{symbols}]'),
            (uppercase, r'[A-Z]'),
            (lowercase, r'[a-z]')
        ]

        # Check constraints
        if all(
            constraint <= len(re.findall(pattern, password))
            for constraint, pattern in constraints
        ):
            break

    return password


if __name__ == '__main__':
    new_password = generate_password()
    print('Generated password:', new_password)
```
It was fun to see this script crank out a 16,384-character random password. Too bad I can't use it on my Netflix account!

## My First Solo Flight

To drive the lesson even further, each module comes with a final project. You're given criteria for a program, and you need to produce a solution entirely on your own that passes a series of pre-determined tests. So far, in my short 2-day journey, this was the most interesting thing I've found.

The first project was to create a Python script that would take a list of arithmetic functions and pretty-print them to the console. There was an added twist that if the function call was flagged 'True' the output also had to include the result of the calculation.

Here's a [link to my solution on my GitHub](https://github.com/stevesansford/free-code-camp/blob/main/python-arithmetic-formatter/artimetic-formatter.py). It's messy, but it works. It passed all the tests. I plan to revisit this challenge in a few months to see what the new solution looks like.

Until then, only 98 more days to go...

