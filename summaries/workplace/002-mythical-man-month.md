The Mythical Man-Month
==========
Essays on Software Engineering
-------------

Frederick Brooks • 1975, 1995 • 293 pages

[Goodreads](https://www.goodreads.com/book/show/13629.The_Mythical_Man_Month)

Author
------------
Frederick Brooks (born April 19, 1931) is an American computer architect, software engineer, and computer scientist, best known for managing the development of IBM's System/360 family of computers and the OS/360 software support package.
Brooks has received many awards, including the National Medal of Technology in 1985 and the Turing Award in 1999.



Introduction
============

This is a classic and appears in many *must-read* lists for better understanding software development.
The book is a list of principles that the author came up with during his long career spent in managing big software projects.
The main idea is that **conceptual integrity** is the most important consideration in software design. It is better to have a system omit certain anomalous features and improvements, but to reflect one set of design ideas, than to have one that contains many good but independent and uncoordinated ideas



Summary
=======

### Man-month unit is a myth

The metric called "man-month", representing the work done by one person in one month, it's based on faulty assumptions such as that "men and months are interchangeable", so project managers can divide work perfectly, anyone can do any job and that no one needs to communicate. Some tasks cannot be divided. Others will require very complex organization and communication processes. When n people have to communicate among themselves, as n increases, their output decreases and when it becomes negative the project is delayed further with every person added.

Group intercommunication formula: n (n − 1) / 2
Example: 10 developers give 10 · (10 – 1) / 2 = 90 channels of communication.

### Conceptual integrity is the most important consideration in system design

> I will contend that conceptual integrity is the most important consideration in system design. It is better to have a system omit certain anomalous features and improvements, but to reflect one set of design ideas, than to have one that contains many good but independent and uncoordinated ideas.

A software product, in order to be user-friendly, must have conceptual integrity, which can only be achieved by separating architecture from implementation. A single chief architect (or a small number of architects), acting on the user's behalf, decides what goes in the product and what stays out. The architect or team of architects should develop an idea of what the product should do and make sure that this vision is understood by the rest of the team. A novel idea by someone may not be included if it does not fit seamlessly with the overall product design. In fact, to ensure a user-friendly product, a product may deliberately provide fewer features than it is capable of. This is a lot harder than it seems and will require something called "sacrifice culture" (https://www.youtube.com/watch?v=_CVfNpg8TrI).

### Why did the tower of Babel fail?

> According to the Genesis account, the tower of Babel was man's second major engineering undertaking, after Noah's ark. Babel was the first engineering fiasco. Why did the project fail? Where did they lack? In two respects: communication, and its consequent, organization. They were unable to talk with each other; hence they could not coordinate. When coordination failed, work ground to a halt. Reading between the lines we gather that lack of communication led to disputes, bad feelings, and group jeal- ousies. Shortly the clans began to move apart, preferring isolation to wrangling

Communication is the most critical component of complex programming projects. A project is like the Tower of Babel: with clear and unified communications, you can reach for the heavens, but if your communications are disrupted, your edifice will crumble.




Conclusion
==========

The book has aged badly. The references, which made sense 15 years ago, no longer hold water, and the most-referenced-project is certainly no longer the way we write software nowadays. I still think there are some good take-aways, expecially the challenge of communications in growing teams and the importance of conceptual integrity.



Bibliography and Resources
==========================
* Peopleware: Productive Projects and Teams
