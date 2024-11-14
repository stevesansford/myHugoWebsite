+++
date = '2024-11-13T10:10:38-05:00'
draft = false
title = 'Day #6: Programming and Coding Are Different'
tags = ['#100DaysOfCoding']
+++
My takeaway lesson from today reinforces the idea that programming and coding are two different things. The terms are often used synonymously, but that's incorrect. Coding is the ability to write instructions a computer can follow, and programming is figuring out what instructions the computer needs to produce the desired output.

I spent part of my morning session finishing the eighth lesson in [FreeCodeCamp's Scientific Computing in Python](https://www.freecodecamp.org/learn/scientific-computing-with-python/) course. It involves creating a "shortest path first" algorithm. I understand enough about Python to code the instructions I was given, but I struggled to comprehend the flow of instructions in the script. 
> I knew what I was writing but not why I was writing it.

Here's the finished code from the lesson:

```
my_graph = {
    'A': [('B', 5), ('C', 3), ('E', 11)],
    'B': [('A', 5), ('C', 1), ('F', 2)],
    'C': [('A', 3), ('B', 1), ('D', 1), ('E', 5)],
    'D': [('C',1 ), ('E', 9), ('F', 3)],
    'E': [('A', 11), ('C', 5), ('D', 9)],
    'F': [('B', 2), ('D', 3)]
}

def shortest_path(graph, start, target = ''):
    unvisited = list(graph)
    distances = {node: 0 if node == start else float('inf') for node in graph}
    paths = {node: [] for node in graph}
    paths[start].append(start)
    
    while unvisited:
        current = min(unvisited, key=distances.get)
        for node, distance in graph[current]:
            if distance + distances[current] < distances[node]:
                distances[node] = distance + distances[current]
                if paths[node] and paths[node][-1] == node:
                    paths[node] = paths[current][:]
                else:
                    paths[node].extend(paths[current])
                paths[node].append(node)
        unvisited.remove(current)
    
    targets_to_print = [target] if target else graph
    for node in targets_to_print:
        if node == start:
            continue
        print(f'\n{start}-{node} distance: {distances[node]}\nPath: {" -> ".join(paths[node])}')
    
    return distances, paths
    
shortest_path(my_graph, 'A', 'F')
```
I was reminded of a Spanish tutor I had some years back. She handed me a page of typed content at our first meeting and asked me to read it. My Spanish at the time was very rudimentary, maybe a 250-word vocabulary, but I could read every word on the page with acceptable pronunciation. I understood none of it. 

> There's a difference between being able to read something (code) and understanding what it means (programming).

Here's a great 8-minute video on YouTube that [explains the Dijkstra Shortest Path algorithm](https://www.youtube.com/watch?v=bZkzH5x0SKU). Watching this first would have helped me understand what the instructions I was writing were trying to do.

I'll [brush up on recursion](https://www.youtube.com/watch?v=ivl5-snqul8) before I tackle the next lesson in this series.

## No Such Thing as a Free Lunch

Unless you're looking for coding lessons. Many of the best ones are free. I found one today, even though I wasn't looking for a new one. When watching the recursion video from Bro Code linked above, I got a recommendation for his [new 12-hour free Python course](https://www.youtube.com/watch?v=ix9cRaBkVe0) posted just two months ago.

I found his recursion explanation concise and easy to follow, so even though I was short on time, I thought I'd put it on and listen to the audio from the car. Turns out that's an interesting way to consume a video tutorial. This Python tutorial is exceptional. I have two key takeaways from the two hours and 38 minutes:

### Audio-Only Might Be a Thing

I've spent enough time studying chess to appreciate the value of visualization. Visualizing the board in your head and following along with moves without is a skill most chess masters have cultivated. It's fascinating to see a grandmaster play [a chess game (or three) blindfolded](https://www.youtube.com/watch?v=xmXwdoRG43U).

>I found myself visualizing the code in my head as it was described in the video. I did not feel I was missing out on anything by not watching the screen.

As I was listening to the audio from this course on my drive today, I found myself visualizing the code in my head as it was described in the video. I did not feel I was missing out on anything by not watching the screen. It was basic stuff being covered, but this might be something I add to my learning process.

### This Course is The Best I've Found

This is entirely subjective, but the BroCode course is exceptional. I've browsed quite a few beginner Python courses as I prepped for this **#100DaysOfCoding** challenge, and so far, this video has shown two interesting things in Python that no other course I've seen explains:

* **Format specifiers**: These would have been handy for the FreeCodeCamp Arithmetic Formatter exercise, although the tests would have failed.
* **The built in ```.help()``` method**: Having a quick reference for methods is useful.

Also, the author is soliciting donations to charity in lieu of personal donations. So far, the course has raised over $8,200 for a children's hospital. What more can you ask for?

I'm going to dedicate a portion of my learning time over the next few days (weeks?) to work through this course in its entirety. I like the style, and I want to experiment further with my visualization exercises. Might be nothing.

We'll find out in 94 more days...