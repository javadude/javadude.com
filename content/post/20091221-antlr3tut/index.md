---
title: ANTLR 3.x Tutorial
aliases:
- /articles/antlr3xtut/index.html
- /articles/antlr3xtut

categories:
- Videos

date: "2009-12-21"

description: Learning to use ANTLR 3x, a Java Parser Generator.

tags:
- java
- parsing
- antlr
- language
- dsl


---

A video tutorial on ANTLR 3.x

<!--more-->

Many folks have asked me to convert my ANTLR 2.x tutorial to ANTLR 3.x. I started doing it and got reaaaaaaaaaalllly tired of typing. Being a bear of very little patience, I decided to go a different route, one which I think will prove even more effective. Go laziness!

This version of the tutorial is video-based. Using Camtasia (the best screen/video capture software in the world!) I've recorded my babbling on while creating a parser for the XL language in ANTLR 3.x in Eclipse.

# Videos

I've uploaded the following videos to vimeo. Each is listed below with a short description. I recommend you watch them in order as they assume knowledge of previous videos.

Vimeo video group: [http://vimeo.com/groups/29150](http://vimeo.com/groups/29150)

## Concepts - What I should have recorded first but forgot until I was 6 steps in...

<iframe src="https://player.vimeo.com/video/8324793" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8324793">ANTLR 3.x Video Tutorial - Concepts</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>


## Prologue - Getting Eclipse set up for ANTLR 3.x development

### Part A: Setting up ANTLR 3.x in Eclipse 3.5
<iframe src="https://player.vimeo.com/video/8001326" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8001326">Setting up ANTLR 3.x in Eclipse 3.5</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part B: Creating and Executing a Grammar in Eclipse

<iframe src="https://player.vimeo.com/video/8015802" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8015802">ANTLR 3.x Creating and Executing a Grammar in Eclipse</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>


## Creating an XL Recognizer

### Part 1: Starting to implement XL
<iframe src="https://player.vimeo.com/video/8137747" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8137747">ANTLR 3.x Tutorial - Part 1</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part 2: Let's check out the generated code
<iframe src="https://player.vimeo.com/video/8138136" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8138136">ANTLR 3.x Tutorial - Part 2</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part 3: Parsing expressions
<iframe src="https://player.vimeo.com/video/8138418" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8138418">ANTLR 3.x Tutorial - Part 3</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

    
NOTE: I noticed I messed up the expression tree a bit. That's what I get for doing this on the fly ;) The tree for 3+2\*6+4 should look like
    
                +
               / \
              +   4
             / \
            3   *
               / \
              2   6
    
### Part 4: Ifs, loops, and comments
<iframe src="https://player.vimeo.com/video/8138774" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8138774">ANTLR 3.x Tutorial - Part 4</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part 5: Procedures and Functions
<iframe src="https://player.vimeo.com/video/8139003" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8139003">ANTLR 3.x Tutorial - Part 5</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part 6: Type declarations, scanner fragments, cleaning up
<iframe src="https://player.vimeo.com/video/8139310" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8139310">ANTLR 3.x Tutorial - Part 6</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>


## Interpreting Expressions

### Part 7: Interpreting expressions while parsing the input
<iframe src="https://player.vimeo.com/video/8377320" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8377320">ANTLR 3.x Video Tutorial - Part 7</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part 8: Interpreting expressions using the GoF Interpreter Pattern
<iframe src="https://player.vimeo.com/video/8377479" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8377479">ANTLR 3.x Video Tutorial - Part 8</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

### Part 9: Interpreting expressions using an ANTLR Tree Parser
<iframe src="https://player.vimeo.com/video/8377605" width="640" height="480" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/8377605">ANTLR 3.x Video Tutorial - Part 9</a> from <a href="https://vimeo.com/user566590">Scott Stanchfield</a> on <a href="https://vimeo.com">Vimeo</a>.</p>


# Video License

[![Creative Commons License](http://i.creativecommons.org/l/by-nc-nd/3.0/us/88x31.png)](http://creativecommons.org/licenses/by-nc-nd/3.0/us/)  
ANTLR 3.x Tutorial by [Scott Stanchfield](http://javadude.com/articles/antlr3xtut) is licensed under a [Creative Commons Attribution-Noncommercial-No Derivative Works 3.0 United States License](http://creativecommons.org/licenses/by-nc-nd/3.0/us/).  
  
Feel free to watch the videos and point friends to them, but you cannot use them as part of any commercial product nor can you create derivative works. Follow the link above to see the complete license text.

# Sample Code

The following are snapshots of the sample code at various points during the tutorial.

*   After part 6 (recognizer): [antlr3xtut-part6.zip](antlr3xtut-part6.zip)
*   After part 9 (expression interpreters): [antlr3xtut-part9.zip](antlr3xtut-part9.zip)

# Software License

All sample code is licensed under the Eclipse Public License, version 1.0.

See [http://www.eclipse.org/legal/epl-v10.html](http://www.eclipse.org/legal/epl-v10.html) for details.

ANTLR itself is licensed under a BSD License:

# ANTLR 3 License  
  
[The BSD License]
Copyright (c) 2003-2008, Terence Parr  
All rights reserved.  

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:  
  
Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.  

Neither the name of the author nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.  

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
