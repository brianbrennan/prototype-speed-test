# prototype-speed-test

To Prototype, or not to Prototype
--------------------------------


Prototype vs closure is an extremely interesting look at Object Oriented Javascript. I'd describe myself as an intermediate level javascript developer, and have been trying to learn as much as I can with this lovely language. I quickly found myself wanting to use Object Oriented Programming as efficiently as possible for my web sites. Though for small sites, the impact of using extremely DRY and abstracted code is probably marginal, for larger applications its plain to see that having your code not be repetitive is a saver of time, amongst other things like bandwidth usage, memory usage, and all the other goodies that should be optimized. While brushing up on my OOP and applying it to JavaScript, I ran accross the interesting difference in instantiating objects in JS: Prototypes. I'm sure many of the developer reading this are thinking what a noob I must be for just learning this concept, but hey, I've never needed it before. So I wanted to learn why it was such a big deal. What was the difference between prototypes and closures in JS as far as speed, memory usage, and other important factors. As it turns out, this is a very commonly asked question on StackOverflow, but I wanted to see for myself what the big hubub was about. So I decided to concoct a test between these two approaches to creating objects. "A simple test on a large scale should do the trick", I thought. So I very quickly jotted down a sketch of a dice rolling script that would take a lot of dice objects, put them into an array, and see the differences between the two. First up, is creating the dice objects in both their flavors.

Writing the Code
----------------

 First up, a Die without prototyping.

	//without prototype
	function Die(sides){
		this.sides =  sides;
		this.roll = function(){
			return Math.floor(Math.random() * 6) + 1;
		}
	}

As it stands here, whenever a new instance of Die is created, the roll method will be completely recreated within object. This creates a lot of memory usage that can be avoided if prototyping was added. So next up, a Die with prototyping.

	//with prototype
	function ProtoDie(sides){
		this.sides = sides;
	}

	ProtoDie.prototype.roll = function(){
		return Math.floor(Math.random() * 6) + 1;
	}

If you're like me and learned a different programming language before JavaScript, this might seem a bit odd. This ProtoDie is in two seperate parts. Doesn't that defeat the purpose of encapsulation? The answer is, not at all. While having custom functions floating separate to your objects would absolutely be bad practice, the use of prototype here is not that. Instead, it is a single instance of the method listed, in this case 'roll', across all of the different ProtoDie objects. Now you might be thinking, "what's the big deal about that"? Let's keep building this simple program and find out. The first thing I tried testing was the speed of instantiation across multiple (and I mean multiple) objects. 

	var dice = [];

	var protoDice = [];


	//make 100 instances of each object
	for(var i = 0; i < 100; i++){
		dice.push(new Die(6));
		protoDice.push(new ProtoDie(6));
	}

Here we simply have two arrays, one for closure dice, and the other for prototype dice. After that we have a loop to make 100 objects in the array of the relative types that use that afforementioned methods. Next, I tried testing the speed of initiatig the roll method on each of these objects.

	//roll once for each instance in dice

	var start = Date.now();

	for(var i = 0; i < 100; i++){
		dice[i].roll;
	}
	var end = Date.now();

	console.log("Rolling Without Prototyping: " + (end - start) + "ms");

	start = Date.now();

	for(var i = 0; i < 100; i++){
		protoDice[i].roll;
	}
	end = Date.now();

	console.log("Rolling With Prototyping: " + (end - start) + "ms");

Well, as it turns out, 100 objects is not enough to get an idea of the speed difference between these two. Modern computers are just too quick for a 100 objects to cause any noticeable difference between the two methods as far as instantiation speed goes. But I figured I might as well up the ante a little and try and see if/when there is a differnce in speed. as it turns out, if you change those for loops to make 100,000 instances of each object, rather than 100, you do get a difference in instantiation speed.

	Rolling Without Prototyping: 7ms
	Rolling With Prototyping: 8ms
	

The fact that it took 100,000 objects for each method to see a noticeable difference means that the difference in speed between these two methods is less than negligeable as far as I'm concerned. But what are some of the other differences between these two approaches to OOJS. 

The Good Results
----------------
The other thing I wanted to test was memory usage. That seems to be the big difference. The fact that there exists only one method using the prototype approach, across all the instances of the specific object, HAS to mean a huge decrease in memory. And so, I tested it. At first, I made the mistake of testing for the size of the objects in my script.The problem with this is that these comparisons just take in to account the size of the objects construct methods, which are the same in both cases. I mistakingly used the script from the StackOverflow answer here http://stackoverflow.com/questions/1248302/javascript-object-size.
So instead, I dove into Chrome's Dev tool, took a heap snapshot, and looked for the difference between these two types of objects. At last, I found the conclusive difference I had been searching for. 

				
	Closure: 	5600 bytes	
	Prototype: 	1600 bytes

That is a serious increase in memory usage between closures and prototypes. Satisfied with finding my answer, I decided to do a bit more research to see what others came up with. I found this comprehensive benchmark here (https://github.com/podefr/benchmarks/blob/master/proto-vs-closure-memory/closure.js), showing that the different in memory usage between prototypes and closures increases dramatically with more methods on the objects. Finding out this size difference was an awesome way to understand more of how prototypes work.

