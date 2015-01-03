---
layout: post
title: Private and Public scopes in Javascript
comments: true
---
As promised in my previous article [Understanding scope in Javascript](http://kamranahmed.info/blog/2015/01/02/understanding-scope-in-javascript/), in this article I will be explaining how can we use OOP constructs such as `private` and `public` access specifiers in Javascript.

From a [previous article of mine on structuring Javascript](http://kamranahmed.info/blog/2014/08/07/how-to-structure-your-javascript/, we all know that how and why should we structure our Javascript code to make it easy to maintain. I described how can we introduce modules in our Javascript code. Let me borrow an example from that article and show you how we decided our modules to look like:

<pre><code class="javascript">
var FeedReader = {
 
    settings: {
        feedItemsCount: 10,
        url: 'http://someurl.com/news/feed/',
        feedListing: $('div#feedListing'),
        loadFeedButton: $('a.loadFeed')
    },
 
    init: function () {
        FeedReader.showErrorIfSourceDead();
        FeedReader.bindUI();
    },
 
    bindUI: function () {
        FeedReader.settings.loadFeedButton.on('click', function ( e ) {
            e.preventDefault();
            FeedReader.fetchFeed();
        });
    },
 
    showErrorIfSourceDead: function () {
        if ( FeedReader.isSourceAlive( FeedReader.settings.url ) === false ) {
            alert("The URL provided can't be used to fetch feed.");
        }
    },
 
    isSourceAlive: function () {
       // ...
    },
 
    fetchFeed: function () {
        // Fetch feed from the `FeedReader.settings.url`
        // Append the feed to the `FeedReader.settings.feedListing`
    }
};
</code></pre>

Structuring your code this way is good, but there is a little *problem* with this approach. As you can see, we don't have any `private` or `public` variables or functions setup, everything that is in `FeedReader` is public and can be accessed by the outer world whether it should be or not be. In this article, I am specifically going to deal with this problem and show you how you can make use of `private`, `public` and also `static` variables and functions. So let's begin.

#Creating a Module
First things first, let's create our *class* (Module) first and we will be gradually extending it as I introduce you to different concepts. I am going to use the word Module to reference our so called Javascript classes. Let's create a `Person` module that is just a simple function. Here is what I mean

<pre><code class="javascript">
// Person Module
var Person = function ( name ) {
    // Do some stuff here    
}
</code></pre>

Above we have created our person module, which has a *constructor* that accepts `name` parameter. And here is how you can instantiate your modules.

<pre><code class="javascript">
// Create a person named Foo
var Foo = new Person('Foo');
// Create a person named Bar
var Bar = new Person('Bar')
</code></pre>

#Private Scope
While writing your Javascript, security should be the one of your top priorities, as we can't afford to make our sensitive variables and functions be publically accessible as they have tendency to make us open to vulnerable attacks. 

You should never expose any unnecessary variables or functions. To do so, we make use of `private` variables and functions. Below is how we implement this privacy in javascript. In our example, let's create some private variables and functions

<pre><code class="javascript">
// Person Module
var Person = function ( name ) {
    
    // Private variables and functions that
    // ..cannot be accessed outside this Module
    var age     = 0,
        isAlive = true,
        name    = name || 'Un-named';
    
    var growOld = function () { 
        age++; 
    }

    var die = function () {
        isAlive = false;
    }
}
</code></pre>

In the above snippet the variables and functions defined are only accessible inside the `Parent` module and cannot be accessed outside. Let's instantiate our module and see if we can access them outside the module or not.

<pre><code class="javascript">
// Create a person named Foo Bar
var FooBar = new Person('Foo Bar');

// TypeError: undefined is not a function
FooBar.growOld();
FooBar.die();

// undefined
console.log( FooBar.isAlive );
</code></pre>

As you can see, neither were we able to access any of the private variables nor any functions.

#Public Scope
In our `Person` module we only have got private variables and functions and there is no way for us to interact with `Person`. To make it more interactive, let's extend our `Person` a bit and add some public variables and functions.

<pre><code class="javascript">
// Person Module
var Person = function ( name ) {
    
    // Private variables and functions
    // ...
    

    // All the properties and methods contained by 
    // ..this object being returned will be public
    // ..and will be accessible in the global scope.
    return {
        passTime: function() {

        },

        speak : function () {

        }
    }
}
</code></pre>

To make some public functions and variables, all you have to do is return an object from the function. The properties and methods possessed by this object will be made public and will be accessible outside the Person module.

#Our Complete Module
Let's extend our example module a bit and see what we have got

<pre><code class="javascript">
// Person Module
var Person = function ( name ) {
    
    // Private variables and functions that only
    // ..other private or public functions may access
    // ..and cannot be accessed outside this Module
    var age       = 0,
        maxAge    = 30,
        maxWeight = 80,
        isAlive   = true,
        weight    = 20,
        name      = name || 'Un-named';
    
    var growOld = function () { 
        if ( age++ >= maxAge ) {
            die();
        }
    }
    var gainWeight = function () { 
        if ( weight++ >= maxWeight ) {
            die();
        }
    }

    var loseWeight = function () { 
        if ( weight-- <= 0 ) {
            die();
        }
    } 

    var die = function () { isAlive = false; }
    

    // All the properties and methods contained by 
    // ..this object being returned will be public
    // ..and will be accessible in the global scope.
    return {
        speak : function () { 
            if ( !isAlive ) {
                alert('Dead man can\'t speak.');
                return;
            }

            alert('Speaking...');
            growOld(); 
        },

        walk : function () { 
            if( !isAlive ) {
                alert('Dead man can\'t walk');
                return;
            }

            alert('Walking'); 
            growOld(); 
            loseWeight(); 
        },

        eat : function () {
            if ( !isAlive ) {
                alert('Dead man can\'t eat');
            }
            gainWeight(); 
        }
    }
}
</code></pre>

You can have a look at the [working demo of the module at jsfiddle](http://jsfiddle.net/41ootg5f/). 

And that's about it. I hope you have gained a good understanding of private and public scopes by now. Also, you should know that there are other ways to achieve the same however the one covered in this article would probably suffice. Plus there are other things such as `prototype`, `priviliged` scope and `static` properties and methods etc to get the most out of the modular pattern that was out of the scope of this article and wasn't discussed, however you should expect that to come soon. If you have any questions, feel free to leave a comment.