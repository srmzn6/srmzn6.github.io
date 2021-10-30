---
title: "Calculating Distance Between 2 Points on the Earth"
date: 2010-10-22 16:10:19.000000000 -04:00
draft: false
tags:
    - Datalogger
    - Distance Calculation
    - GPS
    - Haversine
    - Math
keywords:
    - Coding
    - GPS
---
Out of curiosity I had built myself a GPS Datalogger, I was able to get the raw NMEA data out of it but it only provided information regarding my position and time. I wanted to calculate the velocity of a vehicle at a given location and plot a graph of distance Vs velocity. I used the following calculations to calculate the distance between the two coordinates on the Earth's surface. By dividing that distance with time I was able to calculate the velocity in KMPH at a given location.     
To calculate the distance between two Latitude and Longitude coordinates, I am using the 'Havesine' formula. Haversine formaula calculates the great-circle distance between the two points i.e. the shortest distance given between the two points on the earth's surface. It calculates the "as-the-crow-flies" distance between the points ignoring any terrain.      
Here's the formula (angles are assumed to be in Radians)      

R = Earth's radius = 6,371 Km      
Dlat = latitude2 - latitude1      
Dlong = longitude2 - longitude1      
a = sin²(Dlat/2) + cos(latitude1)*cos(latitude2)*sin²(Dlong/2)       
c = 2.atan2(√a, √(1−a))      
d = R*c      

### Haversine formula (Mathematical explanation)
haversin (d/R) = haversin(θ2 - θ1) + cos(θ1)cos(θ2)havesin(Δλ)      
Where:      

havesine(θ) = sin²(θ/2) = (1−cos(θ))/2      
d is the distance between two points on a sphere       
R is the radius of the sphere      
θ1 is the latitude at point 1      
θ2 is the latitude at point 2      
Δλ is the longitude seperation       

The arguments to the haversine function are assumed to be in Radians.       
so, to calculate the distance we have      

d = R haversine(inverse)(h) = 2Rarcsin(√h)      

Where h = haversin (d/R)
My implementation of the above formula is in Java, I be publishing the code as soon as I get some free time.
I can email the code if anybody wants it urgently.

### References:
1. http://en.wikipedia.org/wiki/Haversine_formula
2. http://www.movable-type.co.uk/scripts/latlong.html
