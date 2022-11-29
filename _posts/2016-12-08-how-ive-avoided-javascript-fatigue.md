---
layout: post
title: How I have avoided "javascript fatigue"
category: articles
tags: [frontend, opinion]
---

I keep hearing about "javascript fatigue" on Hacker News and amongst my friends and I wanted to write something about an exercise anyone can do to avoid it.

### What is javascript fatigue?

The idea behind "javascript fatigue" is that all of the available tools and libraries have somehow made frontend web development a more difficult and frustrating thing to do than in the past. At first, I thought it was just a little inside joke or venting amongst experienced frontend developers, but now I'm starting to see all this once harmless venting has become toxic and harmful.

There are two casualties to this. The first is the open source maintainer who is  burned out because it's not a joke to them. In some cases it's their life's work. Read [this article by James Kyle](https://medium.com/@thejameskyle/dear-javascript-7e14ffcae36c) to see what I mean. We all know about our community's problems with burnout, depression, and suicide. We don't need to do that to people who dedicate so much time and energy into trying to help us.

The second is the new person who is coming into this world thinking this is the norm. They didn't get the privilege to learn all this stuff when frontend was allegedly easier and are now told that their confusion is normal and will probably last forever. It's soul crushing and driving people to quit. Learning this stuff should be fun and easy and it always has been.

### The Exercise

This exercise is aimed at people who are just getting started as well as people who have been doing this for so many years that they forgot you can even still do things this way. Even if you know what is going to happen, you should physically do these steps anyway.

1. Start by opening a text editor. Don't use your editor of choice if you have one. Use what comes with your computer. For example, use TextEdit on Mac, Notepad on Windows. **Do shift+command+T in TextEdit to get into plain text mode**

2. Save the file on your desktop as `page.html`. Make sure the extension is html.

3. Paste the code below into the file

4. Save the file again and double click the file on your desktop

<figure>
{% highlight html %}
<html>
  <body>
    <ul id="counts">
    </ul>
  </body>

  <script>
    var ul = document.getElementById("counts");
    var count = 0;

    setInterval(function() {
      // make a list item <li>123</li>
      var li = document.createElement("li");
      li.appendChild(document.createTextNode(count));

      // append to our list
      ul.appendChild(li);

      // increment counter
      count++;
    }, 1000); // wait one second
  </script>
</html>  
{% endhighlight %}

<figcaption><a href="https://output.jsbin.com/zuvaxej/1" target="_blank">View the results</a></figcaption>
</figure>

Congratulations, you just created a javascript application with no tools! You can just use the things that come with virtually every computer. You even have a debugger.

It's honestly still impressive to me how simple and accessible the web really is as a platform and when I see something like this run I still laugh a little bit. I'd also argue that this is a perfectly fine way to build things for most people. It works on all devices I've tried and I still use lots of useful applications built in this way.

### Let's keep going

You might be thinking that having a library like jQuery would make you more comfortable and the raw vanilla JS API is too much. Instead of setting up `npm`, why not just include it with another script tag. You can even use a fast and free CDN and choose between minified or not. You can choose and freeze the version just like npm would.

{% highlight html %}
<html>
  <head>
    <!-- no need for minified, why complicate it? -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.2.4/jquery.js"></script>
  </head>
  <body>
    <ul id="counts">
    </ul>
  </body>

  <script>
    $(document).ready(function() {
      var $ul = $("#counts");
      var count = 0;

      setInterval(function() {
        $ul.append("<li>"+count+"</li>");
        count++;
      }, 1000);
    });
  </script>
</html>  
{% endhighlight %}

You might be tempted now to reach for a cool new framework like `React`. Keep in mind it would be perfectly fine to continue on with just jQuery. In fact, if you are just getting started, I'd say learn jQuery and build a couple giant spaghetti applications before you bother looking at anything else. But if you are really in the mood to learn it, or maybe need to for a job, React can be a great library to learn and you still don't need any tools!

{% highlight html %}
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.2.4/jquery.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.4.1/react.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.4.1/react-dom.js"></script>
  </head>
  <body>
    <div id="count"></div>
  </body>

  <script>
    var Count = React.createClass({
      render: function() {
        var c = this.props.value;
        return React.createElement("li", null, c);
      }
    });

    var CountList = React.createClass({
      getInitialState: function() {
        return {counts: [0]};
      },
      increment: function() {
        var c = this.state.counts[this.state.counts.length - 1];
        this.state.counts.push(c + 1);
        this.setState({counts: this.state.counts});
      },
      componentDidMount: function() {
        setInterval(this.increment, 1000);
      },
      render: function() {
        var liElements = this.state.counts.map(function(c) {
          return React.createElement(Count, {value: c}, null);
        })
        return React.createElement("ul", null, liElements);
      }
    });

    $(document).ready(function() {
      ReactDOM.render(React.createElement(CountList), document.getElementById("count"));
    });

  </script>
</html>  
{% endhighlight %}

So what about ES6? Maybe you think ES5 is too verbose. Should we start configuring a `.babelrc` now? No need, ES6 is actually a standard that your browser probably already supports. [It's still patchy](http://kangax.github.io/compat-table/es6/) but I'd be willing to bet that all the devices (and most browsers) you own already support it. Let's even add in JSX for fun. It's not supported anymore but we can include a JSX transformer from the CDN and it works fine in this case.

{% highlight html %}
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.2.4/jquery.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.4.1/react.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.4.1/react-dom.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.13.3/JSXTransformer.js"></script>
  </head>
  <body>
  </body>

  <script type="text/jsx">
    class Count extends React.Component {
      render() {
        return <li>{this.props.value}</li>
      }
    }

    class CountList extends React.Component {
      constructor(props) {
        super(props)
        this.state = {counts: [0]}
      }
      increment() {
        let c = this.state.counts[this.state.counts.length - 1]
        this.state.counts.push(c + 1)
        this.setState({counts: this.state.counts})
      }
      componentDidMount() {
        setInterval(this.increment.bind(this), 1000)
      }
      render() {
        let els = this.state.counts.map(c => <Count value={c} />)
        return <ul>{els}</ul>
      }
    }

    $(document).ready(() => {
      ReactDOM.render(<CountList />, document.body)
    })
  </script>
</html>
{% endhighlight %}

Let's take stock of what we have. We have a working React/JSX + ES6 application that works on most of our devices and browsers. Let's point out what we didn't need to accomplish it.

1. Nothing installed or configured on the system
2. No dependency management (npm)
3. No transpiler (Babel)
4. No minification
5. No crazy build system with separate "plugins" (Gulp/Grunt)
6. No node.js server that only serves up static content
7. No IDE
8. No testing frameworks
9. No javascript module tools (e.g. Webpack)

### My thoughts on tools and libraries

Even though you **absolutely can** build great software without tooling, I'm not advocating this is the way everyone should be doing it (though if you are just starting out, I think you should be). What the exercise is meant to do is remind you (or maybe show you for the first time) that all of these tools don't really help you build good, working frontend applications. They can be useful for deploying and maintaining applications, but none of them are required. If your application grows in users and programmers you will likely find you want at least some of these tools. We use some of these at work and couldn't do without them. But add them one at a time and understand how they work and why they were made. Don't think that you have to tame a Gruntfile just to try React for instance.

The beauty of doing it this way is it's really easy to add these tools later. I'm not kidding. While most things are easier to learn from the top down, I think the unique thing about the web right now is **it's probably actually easier to learn from the bottom up**. Everytime you just accept that you need one of these tools you are missing out on the fact that it doesn't really do much for you and is probably just getting in your way at the moment.

### Where to go from here

I don't think that reading an html file straight from your desktop as we did in the exercise is a pleasant way to develop but I think there are some tools that come close in the freedom they give you. I like to start prototyping my idea in something like [jsbin](http://jsbin.com/). It supports things like dependencies,  versioning, and even JSX. You can immediately share the url with a friend for feedback or a device you want to test. You can literally copy and paste it out of there later. Don't waste your time setting up a node server and some wacky Gulp plugins for something that no one but you may ever see. You'll be so much happier, your system will be leaner, and you'll understand every piece of it.
