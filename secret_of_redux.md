# Secret of Redux

When I wandered on [Zhihu](https://www.zhihu.com)(Quora for Chinese) a few month ago,
An interesting question came into my view:

> What is Redux? Can anyone explain it in "human" Language?
> 
> _The question [here](https://www.zhihu.com/question/41312576),
> (in Chinese)_

Surely, it's hard to explain such a domain specialized tool like Redux clearly,
and avoid using tons of technical terminology.

However, two months have passed and __even no satisfactory technical answer appeared__.

The only acceptable piece is a translation work for [Redux and The Command Pattern](https://medium.com/@abhiaiyer/the-command-pattern-c51292e22ea7#.gml3ufwnf).
But unfortunately, the article stops with comparing Command Pattern and Redux.
The author didn't write about the idea behind Command Pattern, and why it works.

> You can you up. -- By some anonymous web users in China.
> 
> _(it means "stop criticising unless you can do better")_

Here in this article, I'll try to answer these questions:
* What kind of problems does Redux try to solve?
* What's the idea behind Redux?
* How it works?

__But it's beyond my ability to explain it non-technically.__

If you're expecting a non-technical explanation, 
maybe you can wait for my rewriting this article years later (not promised).
Though Redux might be out-of-fashion at that time,
I'm sure its philosophy will keep working in other tools.  

## My college homework and automata

In my college years in NJU, 
my first programming homework was to replicate a tank war game in Java.
(I don't want to call a 10-line Fibonacci calculator a homework)

It was damn nightmare for a freshman who. 
But I finally worked out a solution to display the game with Java SWT.

I designed the game merely in the following psuedo code:
```Java
class Tank {
  int x;
  int y;
  int speed;
  string direction;
  //...
}

class Bullet {
  int x;
  int y;
  int speed;
  string direction;
  
  //...
}

class Map {
  Array[][] matrix
  
  //...
}

class Game{
 List tanks;
 List bullets;
 Map map;
 //...
}
```

- Every 100 milliseconds, I recalculate all the tanks and bullets' position,
and draw them on the map.
- At the same time, it updates each tank's position and direction according to its category.
- Then 100 milliseconds later, I go back to step 1, to recalculate and repaint.

