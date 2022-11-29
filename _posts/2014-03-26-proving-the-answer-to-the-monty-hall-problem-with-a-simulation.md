---
layout: post
title: Proving the answer to the Monty Hall problem with a simulation
category: articles
tags: [statistics, python, numpy]
---

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/Xp6V_lO1ZKA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>
<p></p>

The [Monty Hall Problem](https://en.wikipedia.org/wiki/Monty_Hall_problem) is a well known statistical paradox. Even knowing the mathematics behind it, it can be difficult to accept that switching doors will change your odds from 33.3% to 66.6%. There are plenty of ways to prove it mathematically but why not see it in action? We could play out the game with 2 people, say *n* times. We could do one round where the player does not switch their choice, then we could do another round where they do switch. At the end we could tabulate the win percentage and if *n* was sufficiently large, the percentages should match up to theory. Or we could just write some python to play this game for us.

### The Code

This code requires [numpy](http://www.numpy.org/) so if you want to run it you will need numpy installed. First let's import it into a namespace *np*:

{% highlight python %}
import numpy as np
{% endhighlight %}

Now we need a function which can generate *n* prizedoors where *n* is the number of simulations we want to run. We are just going to use an integer in the set {0, 1, 2} to represent the *index* of the door. Let's just generate a numpy array of evenly distributed, random choices:

{% highlight python %}
def simulate_prizedoors(n):
    return np.random.random_integers(0, 2, size=n)
{% endhighlight %}

We can do the same thing to simulate the player's guesses:

{% highlight python %}
def simulate_guesses(n):
    return np.random.random_integers(0, 2, size=n)
{% endhighlight %}

Now we need a slightly more complicated function to generate the "goat door". This is the door that the host chooses to reveal (which is always a goat). This function takes the prizedoors and the guesses as arguments, then returns the first door that isn't the guess or the prizedoor.

{% highlight python %}
def goatdoors(guesses, prizedoors):
    return [np.setdiff1d([0, 1, 2], [prizedoors[i], guesses[i]])[0] for i in range(len(prizedoors))]
{% endhighlight %}

We now need one more function which can simulate the player switching their guess. This does a very similar thing to *goatdoors*. It takes the guesses and the goatdoors and chooses the first door that is neither of those:

{% highlight python %}
def switch_guess(guesses, goatdoors):
    return [np.setdiff1d([0, 1, 2], [guesses[i], goatdoors[i]])[0] for i in range(len(guesses))]
{% endhighlight %}

And one more function to take the results of the simulations and calculate the percentage of wins. If this function looks a bit naive it's because my python is rusty:

{% highlight python %}
def win_percentage(guesses, prizedoors):
    count = 0
    for i in range(len(guesses)):
        if guesses[i] == prizedoors[i]:
            count += 1
    return 100.0 * count / float(len(guesses))
{% endhighlight %}

Let's run 10,000 simulations!

{% highlight python %}
N = 10000
{% endhighlight %}

{% highlight python %}
prizedoors = simulate_prizedoors(N)
guesses = simulate_guesses(N)

print "Wins without switching %f" % win_percentage(guesses, prizedoors)
#=> prints 33.3% +- 1.0%

prizedoors = simulate_prizedoors(N)
guesses = simulate_guesses(N)
goatdoors = goatdoors(guesses, prizedoors)

print "Wins with switching: %f" % win_percentage(switch_guess(guesses, goatdoors), prizedoors)
#=> prints 66.6% +- 1.0%
{% endhighlight %}

I also have a [gist of the full code](https://gist.github.com/bhelx/9164056).

