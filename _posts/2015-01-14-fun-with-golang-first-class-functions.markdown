---
layout: post
title:  "Fun with Golang First Class Functions"
date:   2015-01-25 16:56:56
categories: software
---
I've been trying to beat my wife at [Scramble with Friends][scramble-with-friends], one of those Zynga Android games, since Christmas time when my brother-in-law introduced us to the game. It was strangely addicting, and stranger yet when I couldn't beat her. Since then, I've been working on relentlessly beating my wife. Strange when you put it that way...

After trying to do it the good old fashioned way, practice and/or a dictionary, I realized it would be much more fun to make a bot. So I thought long and hard about what language I should use. By long and hard I mean I cded to my Go directory and got to work. 

I've been playing around with Go for a couple months now, but I had only made a couple basic web apps. As suspected, it was much more fun to dive into a real problem!

If you haven't heard of Scramble with Friends, it's a social [Boggle][boggle] rip off. Check out the full rules in the link, but the game is simple. You have a 4x4 board of letters and you try and make words. You can start at any letter on the board and go in any direction. You just can't reuse a letter along the way. Nice, simple, computer likey.

I'll go into my full solutions with [tesseract-ocr][tesseract] and [Android's monkeyrunner][monkeyrunner] in later posts. But for this post, let's say we have a Game struct with the board represented in a 2d slice of strings.

{% highlight go %}
type Game struct {
    Board      [4][4]string
    // More here
} 
{% endhighlight %}

We also have a simple Position struct and a Segment struct representing current board positions and the partial (maybe complete) word those positions make up

{% highlight go %}
type Position struct {
    Row, Column int
}

type Segment struct {
    Positions []Position
    Word      string
}
{% endhighlight %}

To traverse the board and try and form words, we need to move around. My first approach was simple, but I felt miserable as I typed it.

{% highlight go %}
currentPosition := wordSegment.currentPosition()
nextPosition := Position{currentPosition.Row, currentPosition.Column - 1} // Left
if nextPosition.inBounds() && !wordSegment.positionAlreadyPresent(nextPosition) {
    // make new word segment
    // continue recursion
}
nextPosition := Position{currentPosition.Row - 1, currentPosition.Column - 1} // Left-Up
if nextPosition.inBounds() && !wordSegment.positionAlreadyPresent(nextPosition) {
    // make new word segment
    // continue recursion
}
// Check other 6 surrounding positions
{% endhighlight %}

Of course I would have factored out the if into a function, but you still have to figure out where to go for the next statment. I had done this before in programming competitions and it had never felt right. That's when I remebered an awesome difference between Go and those languages I had used. We have first class functions!

{% highlight go %}
var Surroundings []func(Position) Position = []func(Position) Position {
    func(p Position) Position { return Position{p.Row, p.Column - 1} },     // Left
    func(p Position) Position { return Position{p.Row - 1, p.Column - 1} }, // Left-Up
    func(p Position) Position { return Position{p.Row - 1, p.Column} },     // Up
    func(p Position) Position { return Position{p.Row - 1, p.Column + 1} }, // Right-Up
    func(p Position) Position { return Position{p.Row, p.Column + 1} },     // Right
    func(p Position) Position { return Position{p.Row + 1, p.Column + 1} }, // Right-Down
    func(p Position) Position { return Position{p.Row + 1, p.Column} },     // Down
    func(p Position) Position { return Position{p.Row + 1, p.Column - 1} }, // Left-Down
}

currentPosition := wordSegment.currentPosition()
for _, move := range Surroundings {
    nextPosition := move(currentPosition)
    if nextPosition.inBounds() && !segment.positionAlreadyPresent(nextPosition) {
        // make new word segment
        // continue recursion
    }
}
{% endhighlight %}

I thought this approach was much cleaner. What do you think? Is this idiomatic Go? Is there a more elegant way to solve these kinds of problems?

[scramble-with-friends]: https://play.google.com/store/apps/details?id=com.zynga.scramble&hl=en
[boggle]: http://en.wikipedia.org/wiki/Boggle
[tesseract]: https://code.google.com/p/tesseract-ocr/
[monkeyrunner]: http://developer.android.com/tools/help/monkeyrunner_concepts.html
