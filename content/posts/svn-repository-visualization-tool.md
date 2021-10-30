---
title: "Svn Repository Visualization Tool"
date: 2011-01-12 14:58:50.000000000 -05:00
draft: false
keywords:
    - Coding
    - Technology
tags:
    - Gource
    - SVN
    - visualization tool
---
![Grouce!](/static/images/svn.jpg)

I love graphs and charts ( not the ones used by the MBA people) and was looking for a tool which could give me a snapshot or a timeline of the events for our repository we use at my work. My search ended when I stumbled upon a tool called [Gource](http://code.google.com/p/gource/) while browsing [sparkfun.com](http://www.sparkfun.com/).
Let me admit one thing, this tool is the coolest I every played with and its awesomeness brought me on my knees. Basically [Gource](http://code.google.com/p/gource/) is a open source software version control visualization tool for linux, Mac and Windows. It works best with Git but then there are some cool folks [Bitshifter](http://bitshifter.wordpress.com)
who have written python script to make Gource work with SVN.     
following are the steps to use Gource with SVN on windows, I am documenting them so that I don't forget them:    
You need     


1. Gource [download](http://code.google.com/p/gource/downloads/list)
2. Python (2.5 or later) [download](http://www.python.org/download/)
3. [svn-gource.py](http://gource.googlecode.com/files/svn-gource-1.2.tar.gz) 
	

Unzip and install Gource and Python.       
Go to your SVN trunk (or branch) - _my-svn-project_       
`cd my-svn-project`   

Get SVN log     
`svn log --verbose --xml > my-project.log`      

Run the python script to convert svn logs to Gource friendly log      
`python svn-gource.py --filter-dirs my-project.log > my-project-gource.log`     

Display the results      
`gource --log-format custom my-project-gource.log`    

By default the display speed is 10 seconds per day, you can speed it up by using the option `-s` (e.g 0.1 is faster, 100 is slower)
e.g.      
`gource -s 0.1 --log-format custom my-project-gource.log`

