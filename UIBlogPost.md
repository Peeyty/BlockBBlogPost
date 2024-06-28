---
title: "Creating a game UI framework"
preview_title: "UI"
layout: default
---

![Buas Logo](assets/Logo_BUas_RGB.png)


# Creating a game UI framework

### The main feauters of my project

I am a second year Buas student and in this blog post I'm going to explain the text rendering feature of my UI framework project.
For this project I was given a basic c++ engine called 'bee'. This engine is capable of rendering 2d and 3d models, handeling user input and more. 
It also has basic debugging features like keeping track of time between frames and having an inspector to view ECS entities and their properties.
I added a basic UI framework to this engine that is capable of displaying UI windows and elements and that exposses most variables related to these to the inspector.
This allows a designer to easliy moddify windows and elements created by a programmer to make their game UI look exactly how they want it to look.
The UI itself has multiple features like text rendering, image rendering, interactable checkboxes and buttons that can support custom functionality. 
These are the main features of the project. In this blog post I will explain how I handled text rendering as I believe this is the most interesting and complex feature of the project.

### Text rendering

Text rendering can be complicated. In order to get good looking text on screen you want to load a font and then display letters using that font in the correct place. This might sound quite simple but there is quite a lot of hidden complexity in these simple steps.
For example, what is the correct place of a letter in a piece of text? The text as a whole of course needs the right placement on the screen, but an individual letter also needs to take into account its placement compared to the other letters.
This is complicated by the fact that different letters are often placed differently. There are letters that extend beneath the bottom of a sentence, like the small g. 
There are also cases where two specific letters next to each other need a specific different distance between them. For example the letters AV next to one another look better when they are closer together than when you simply start drawing the V immediately after the A.
The process of properly distancing letter combinations next to each other is called kerning. Hopefully this example conveys the complexity hidden in text rendering.
In order to help me render text I used a library called stb_truetype. This library allows me to load fonts from a font file and helps me handle some of the information that the font file contains, like the kerning between different letters.
Using stb_truetype I load all symbols, like the letters, from the font and store them in a big texture saved using opengl. This way when I need to draw a letter I can copy it from this texture using the character index and draw it to the screen.
I store the symbols at a high scale to ensure a nice resolution even when scaling up the letters on the screen. It is also possible to store a texture of all symbols for each scale you use to get an appropriate resolution for the correct size.
When loading the font I also store the extra data that the font contains and that I need later like the kerning. I also store information about the texture I created to contain all symbols. This is information like it's size.

In order to actually render text we need to use and interpet the data we just stored correctly. When rendering text you actually render each symbol in that text one after the other.
So first, how do you render an individual symbol? Most importantly we need to know what symbol we're rendering. In my case when I talk about rendering text I mean I'm rendering a c++ string. A string is an array of characters, characters being a single symbol.
Each character has what's called an ASCII value. This is a way of representing symbols as numbers. For example the number 97 represents a lowercase a. When rendering a string we use these values to see which symbol we actually want to render.
We look at the ASCII value of the character in the string we want to render. When we stored the symbols of the font in a texture we stored them in the ASCII order. Thus by looking at the characters value we know where in the texture this symbol is stored.
An important detail to remember is that the first 32 ASCII values are not actually printable symbols, so when we stored the font we did not store these values either. Therefor you should subtract 32 from the ASCII value to determine the actual place of the symbol in the font texture.
Now we have all the information needed to determine what to draw. We stored a font texture that contains the symbol we want to draw and we know were in the texture this symbol is using the characters ASCII value and the information we stored about the texture.
Now in order to actually draw this symbol we also need to know where to draw it. For this we need quite a bit of information. First of all we need an origin to start drawing from. This is determined by where on the screen you eventually want the text to end up on.
In my program this origin is passed into the function that renders the text. This value should be adjusted to line up with where you want the text to be. Once we have the origin we need the actual placement of the symbol compared to the origin. 
Luckily this information is stored inside the font and therefor stb_truetype gives us access to it. First we need the bearing. The bearing determines the distance between the origin and the upperleft corner of where to draw the symbol.
We can then use the size of the symbol, again gotten using stb_truetype, to determine the exact position of the rectangle on which we draw the symbol using opengl. This is everything required to draw the first symbol, but we need to prepare some information for the next symbol.
We again use stb_truetype, this time to get the kerning for the next symbol. We also get the advance, which helps determine the distance to the next letter when combined with the kerning value.
We now take our origin and increase the horizontal value using the advance and the kerning. This new value becomes the origin for the next symbol, which we render in the same way as the previous one. If we repeat this for every character in our string we will have succesfully rendered our text.

### Outro

Hopefully reading through this blogpost you have learned something about text rendering. Text rendering can be a very complicated topic, so I hope I did it justice and simplified some of the concepts involved.
I you want a more code based explanation of text rendering I highly recommend reading through the learnopengl article on the topic, which uses a library called freetype. This article can be found on learnopengl.com.
