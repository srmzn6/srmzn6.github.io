---
title: "Breaking Linear Congruential Generator"
date: 2012-03-23 09:21:26.000000000 -04:00
draft: false
tags:
    - Breaking LCG
    - Linear Congruential Generator
    - Math
keywords:
    - Coding
    - Cryptography
    - Security
---
Recently I came across [Linear Congruential Generators](http://en.wikipedia.org/wiki/Linear_congruential_generator) (LCG) while taking an online course in Cryptography. Initially it looked like a cute little method to generate pseudo random numbers (PRN), which was simple and elegant but as it turns out it has been broken, pretty badly broken. The class did not go into the details of how it was broken and left it as a mental exercise for students. I will (hopefully) try to demonstrate a way to break it. There could be many other ways to do it; this is using [George Marsaglia](http://en.wikipedia.org/wiki/George_Marsaglia) way. He was the one to expose the "Crystalline" nature of multiplicative generators. For curious readers name of the paper is [RANDOM NUMBERS FALL MAINLY IN THE PLANES](http://www.pnas.org/content/61/1/25.full.pdf). A lot (almost all) of what I will be talking here is borrowed from this paper and other references noted at the end of the post.

_So, what is an LCG ?_   
LCG can be defined as:   
`X(n-1) = (aX(n) + c) mod p`

Where,  
X(n) is a sequence of pseudo random values.   
p is modulo defined as 0 < p     
a is the multiplier defined as 0 < a < p    
c is the increment 0 <= c < p ( if c = 0 the LCG is called Multiplicative Congruential Generator)    

LCG is seeded by the 'initial value' or 'start value' X(0) where 0 <= X(0) < p    
As you can see from the above definition, the current state is used to determine the next state of the LCG.    
Due to the "Crystalline" (see [animation](http://www.cs.pitt.edu/~kirk/cs1501/animations/Random.html)) nature of these generators we can successfully predict the next sequence of random number and calculate the values a,c and p in the above defined equation by just observing few pseudo random values generated by the generator, let’s see how we can achieve this.   
In this example, I am using the LCG used in one of the programming assignments by Stanford Cryptography online course[4] by Prof. Dan Boneh[5].

```
import random
# Linear Congruential Generator
P = 295075153L

class WeakPrng(object):
    def __init__(self, p):   # generate seed with 56 bits of entropy
        self.p = p
        self.x = random.randint(0, p)

    def next(self):
        self.x = (2*self.x + 5) % self.p
        return self.x  

prng = WeakPrng(P)
for i in range(1, 10):
  print "output #%d: %d" % (i, prng.next())
```

The LCG in the following program is   
x = (2*x +5) mod (295075153) .... (I)   
So,   
a = 2   
c = 5 and   
p = 295075153   
Let’s assume we have no knowledge of a,c and p and initial seed for x was chosen randomly (ah, right !). What we do have is the following output sequence generated by the above LCG equation (I).    

LCG output:    
output #1: 54294923    
output #2: 108589851     
output #3: 217179707     
output #4: 139284266    
output #5: 278568537     
output #6: 262061926     
output #7: 229048704     
output #8: 163022260     
output #9: 30969372     

The task here is to find out a,c and p values which would help us predict the next output for this sequence.   
The way I was able to get this to work was using a determinant of a 2x2 matrix, a 3x3 matrix would also work but I haven't tried it.   

Let’s create a 2x2 matrix of above output such as:    
| output(i)-output(1)    output(i+1)-output(2) |    
| output (j)-output(1)   output(j+1)-output(2) |    

The absolute value of the determinant of the above matrix is the volume of a parallelepiped determined by the three output points in the plane (remember the crystalline structure).    
The code below will take space separated pseudo random values and calculate the determinant   
 
e.g. for 7 values generated by the above LCG x(1), x(2), x(3), x(4), x(5), x(6), x(7)    
input = 54294923   108589851   217179707   139284266   278568537   262061926   229048704    

The program calculates determinants:  

d(2,3), d(3,4), d(4,5), d(5,6)    
[note: no error checking done in the program, if you get ‘IndexError: list index out of range’ enter one more pseudo random number]     
d(2,3) = 16021084186723984    
d(3,4) = 25078243389094479    
d(4,5) = 25078243389094479    
d(5,6) = 4870690766336483    

```
import sys

def calc_det(i,j,X):
	""" Calculate the values for the matrix[lattice] """
	a1 = X[i] - X[0]
	b1 = X[i+1] - X[1]
	a2 = X[j] - X[0]
	b2 = X[j+1] - X[1]

	""" Calculate the determinant """
	det = a1*b2 - a2*b1
	return abs(det)

def main():
	print "Calculate determinant 2X2"
	print "Enter the random number list whose determinant you want (space seperated) "
	s = raw_input()
	X = [long(i) for i in s.split(' ')]

	""" calculate determinant for d(1,2) """
	print calc_det(1,2,X)

	""" calculate determinant for d(1,2) """
	print calc_det(2,3,X)

	""" calculate determinant for d(1,2) """
	print calc_det(3,4,X)

	""" calculate determinant for d(1,2) """
	print calc_det(4,5,X)

if __name__ == "__main__":
	sys.exit(main())
```

Because the points lie on the lattice of the structure, the volume must be multiple of unit-cell volume, which is
m^(n-1) in n dimensions.    
So, the modulus of the generator is the GCD of all the possible d(a,b) values. We need not find all the d(a,b) values only few would suffice.
Then,  


p = GCD(d(2,3),d(3,4),d(4,5), d(5,6))     
Following, is the python script to find GCD of multiple numbers.    

```
import sys
import math

def GCD(a,b):
	""" Euclidean Algo"""
	a = abs(a)
	b = abs(b)
	while a:
			a,b = long(b%a),a
	return b

def main():
	print 'Enter the numbers whose factors you want seperated by spaces'
	s = raw_input()
	list = [long(i) for i in s.split(' ')]
	#list = [8, 24, 12]
	soln =  reduce(GCD, list)
	print soln

if __name__ == "__main__":
	sys.exit(main())
```

Feeding the above d(2,3), d(3,4), d(4,5),d(5,6) values to the program we get   

p = 295075153    
Which turns out to be the p that we started with !     

Once we have p, and the initial sequence, it is not at all difficult to find a and c by using equation (I) which is left as an exercise for the reader.      

### References:
1. http://www.pnas.org/content/61/1/25.full.pdf
2. http://en.wikipedia.org/wiki/George_Marsaglia
3. http://en.wikipedia.org/wiki/Linear_congruential_generator
4. https://www.coursera.org/crypto/auth/welcome
5. http://crypto.stanford.edu/~dabo/
6. http://www.math.niu.edu/~rusin/known-math/99/LCG

Some of you might frown on my inefficient Python coding skills. My apologies, this is the first time I have written a python code (an awesome learning experience) and this post is about LCG and not intended to demonstrate my Python skills.
