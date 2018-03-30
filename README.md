# TouchDesigner Process Managment
A look at how you can launch another process from TouchDesigner with Python, and how you can kill that process.

## Contributing Programers / Artists ##
**Matthew Ragan** |[ matthewragan.com ](http://matthewragan.com)  

## TouchDesigner Version ##
All work in this repo is being done in TouchDesigner 099. 

## Overview
Big picture, the idea here is that at some point you'll need to split up the work of a single project into multiple processes. This happens for lots of reasons - maybe you want to break your control interface out from your output elements, or maybe you want to start up another tool you've built - you name it, there are lots of reasons you might want to launch another process. 