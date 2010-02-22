QR
------

**QR** is a Python object for creating and working with **queue data structures for Redis**. Redis is particularly suited for use as a queue, and QR makes it absurdly easy to implement one in Python. QR works best for (and simplifies) the creation of **bounded queues**: queues with a defined size of elements. 


Quick Setup
------------
You'll need [Redis](http://code.google.com/p/redis/ "Redis") itself, and the current Python interface for Redis, [redis-py](http://github.com/andymccurdy/redis-py "redis-py"). Put **rq.py** in your PYTHONPATH and you're all set.

**rq.py** also creates an instance of the redis-py interface object. You may already have instantiated the object in your code, so you'll want to ensure consistent namespacing. You can remove this line of code, modify the namespacing, or adjust your existing namespacing -- whatever works best for you.


QR as Abstraction
------------------

A **queue** is a simple data structure. You **push** values to the back of the queue and **pop** values from the front. 

You can implement two kinds of queues with QR: 

* **Sized queue**: once the queue reaches a specified size of elements, it will atomically pop the oldest element.

* **Unsized queue**: the queue can grow to any size, and will not pop elements unless you explicitly ask it to.

Consider a use case like this: you may have any number of comments on a blog post, but you only want to display the most recent 10. You can use a sized queue, and pop the older comments as new ones come in. (We'll do that exact thing in an example at the end of this README).

Create the Queue & Basic Push and Pop
-------------------------------------

**rq.py** is single-file Python module. It includes a single class, **Rq**. To create a new queue, just create an instance of the Rq class:

* A first-position **key** argument is required. It's the Redis key you want to be associated with the queue.
* A second-position **size** argument is optional. Without a size argument you get an unsized queue. With a specified size, you get a sized queue.

Cool, let's create a version of The Beatles that, rather ahistorically, has just three members. Start your Redis server, and now:

	>> from rq import Rq
	>> beatles_queue = Rq('Beatles', 3)

You are now the owner of an QR object (with the variable name 'beatles_queue'), associated with the Redis key 'Beatles'. The RQ object has a specified size of 3 elements. Let's push some elements:

	>> beatles_queue.push('Ringo')
	FOR KEY: 'Beatles'
	PUSHED: 'Ringo'

	>> beatles_queue.push('Paul')
	FOR KEY: 'Beatles'
	PUSHED: 'Paul'

	>> beatles_queue.push('John')
	FOR KEY: 'Beatles'
	PUSHED: 'John'

	>> beatles_queue.push('George')
	FOR KEY: 'Beatles'
	PUSHED: 'George'
	POPPED: 'Ringo'

Since the queue was **capped at three elements**, the addition of 'George' resulted in a pop of the first element (in this case, 'Ringo'). Sorry, Ringo, you're out of the band.

You can utilize **pop** at anytime. To pop the oldest element, just do this:

	>>beatles_queue.pop()
	FOR KEY: 'Beatles'
	POPPED: 'Paul'

Return the Values
-----------------

Let's return some data from the queue! QR includes two return-style methods: **elements** and **elements_as_json**. 

* Call **elements**, and you'll get back a Python list. 

* Call **elements_as_json**, and you'll get back the list as a JSON object.

For example:

	>>beatles_queue.elements()
	['John', 'George']

	#Let's bring Ringo back into the band
	>> beatles_queue.push('Ringo')
	FOR KEY: 'Beatles'
	PUSHED: 'Ringo'

	#The elements method will return the updated list
	>>beatles_queue.elements()
	['Ringo', 'John', 'George']

	>>beatles_queue.elements_as_json()
	'['Ringo', 'John', 'George']'


A Real-World Example: Blog Comments
-----------------------------------

Imagine you have a Django view that handles blog comments. You have an arbitrary number of blog comments. You want to return only the most recent ten blog comments to your template. Skipping to the relevant parts, it's as easy as this:

	#The recent comments are represented as a queue with 10 elements
	most_recent_comments = Rq('recent_comments', 10)

	#The view has receieved a new comment from a HTML form!
	new_comment = 'No way man, LaForge was the best character!'

	#Add the comment to your comment queue. If there are already 10 comments, the oldest one gets popped.
	most_recent_comments.push(new_comment)

	#Create a list of the most recent 10 comments.
	comments_for_template = most_recent_comments.elements()

	#Now send the comments list back to your template. In the template, you could loop through it and you're done.
	render_to_response ('blog.html', comments_for_template=comments_for_template)

	
MIT License
------------

Copyright (c) 2010 Ted Nyman

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


	


