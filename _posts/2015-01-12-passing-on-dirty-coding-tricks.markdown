---
layout: post
title:  "Passing on Dirty Coding Tricks"
date:   2015-01-12 16:56:56
categories: software 
---
I hate new year resolutions, so my first post being so close to the new year is just a coincidence. I do want to write more. What better way than to let someone else write for me. Just pass some knowledge along.

I saw this link on HN, [Dirty Coding Tricks][dirty-coding-tricks], about programmers sometimes just having to get stuff done. My favorite exceprt is below, but you should read the whole thing.

>I was fresh out of college, still wet behind the ears, and about to enter the beta phase of my first professional game project -- a late-90s PC title. It had been an exciting rollercoaster ride, as projects often are. All the content was in and the game was looking good. There was one problem though: We were way over our memory budget.
>
>Since most memory was taken up by models and textures, we worked with the artists to reduce the memory footprint of the game as much as possible. We scaled down images, decimated models, and compressed textures. Sometimes we did this with the support of the artists, and sometimes over their dead bodies.
>
>We cut megabyte after megabyte, and after a few days of frantic activity, we reached a point where we felt there was nothing else we could do. Unless we cut some major content, there was no way we could free up any more memory. Exhausted, we evaluated our current memory usage. We were still 1.5 MB over the memory limit!
>
>At this point one of the most experienced programmers in the team, one who had survived many years of development in the "good old days," decided to take matters into his own hands. He called me into his office, and we set out upon what I imagined would be another exhausting session of freeing up memory.
>
>Instead, he brought up a source file and pointed to this line:
>
>static char buffer[1024\*1024\*2];
>
>"See this?" he said. And then deleted it with a single keystroke. Done!
>
>He probably saw the horror in my eyes, so he explained to me that he had put aside those two megabytes of memory early in the development cycle. He knew from experience that it was always impossible to cut content down to memory budgets, and that many projects had come close to failing because of it. So now, as a regular practice, he always put aside a nice block of memory to free up when it's really needed.
>
>He walked out of the office and announced he had reduced the memory footprint to within budget constraints -- he was toasted as the hero of the project.
>
>As horrified as I was back then about such a "barbaric" practice, I have to admit that I'm warming up to it. I haven't gotten into the frame of mind where I can put it to use yet, but I can see how sometimes, when you're up against the wall, having a bit of memory tucked away for a rainy day can really make a difference. Funny how time and experience changes everything.
>
>\- *Noel Llopis* from [Dirty Coding Tricks][dirty-coding-tricks]

[dirty-coding-tricks]: http://www.gamasutra.com/view/feature/4111/dirty_coding_tricks.php?print=1
