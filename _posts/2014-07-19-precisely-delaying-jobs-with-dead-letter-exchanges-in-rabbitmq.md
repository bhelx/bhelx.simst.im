---
layout: post
title: Precisely Delaying Jobs With Dead-Letter Exchanges in RabbitMQ
category: articles
tags: [rabbitmq, node]
---

Everyone who is familiar with [RabbitMQ](http://rabbitmq.com/) knows it's a great tool to execute jobs that may happen out of band of some normal user process. Typically you want to execute these jobs right away, such as sending some kind of notification email, but there may be times in which you want the execution of these jobs to be delayed by specific times set by the client. RabbitMQ does not support this directly, but I encountered an interesting trick using [Dead-Letter Exchanges](https://www.rabbitmq.com/dlx.html) to achieve this functionality.

### How It Works

Instead of publishing directly to an exchange as we normally would, the producer creates a new, unique queue and publishes it's message to it. In our code we are calling it `send.later.<timestamp>`.

When we create this queue, we set some special options:

{% highlight javascript %}
{
    "x-dead-letter-exchange" : "immediate"
  , "x-message-ttl"          : 5000  // 5 seconds
  , "x-expires"              : 10000 // 10 seconds
  , "x-dead-letter-key"      : "right-now-queue"
}

{% endhighlight %}

The `x-message-ttl` is a time in milliseconds that you want to delay the job by. This can be unqiue per message. The `x-expires` is the time in milliseconds you wish for that temporary queue to exist. The `x-dead-letter-exchange` is the exchange you wish for the message to be published to after it expires. So, we are sortof doing a semantic hack here. Instead of actually expiring the message we are actually activating it on expiration. `x-dead-letter-key` is the routing key to be used when publishing into the `immediate` exchange.

We have previously created an exchange called `immediate` in which the consumers are listening to run jobs from immediately. When the message `expires`, it will be thrown into this exchange.

### The Code

The code is written in node.js and uses [node-amqp](https://github.com/postwait/node-amqp) to speak to RabbitMQ. It is comprised of two parts (a producer and a consumer) which I have written into two separate programs.

#### The Producer (producer.js)

{% highlight javascript %}
var amqp         = require('amqp')
  , conn         = amqp.createConnection()
  , PUBLISH_RATE = 1000 //every second
  , count        = 1
  ;

conn.on('ready', function() {
  console.log('ready');

  conn.exchange('immediate', options={
    durable: true,
    autoDelete: false
  }, function (exchange) {
    setInterval(function() {
      var key = "send.later." + (new Date().getTime()).toString();
      conn.queue(key, {
        arguments: {
            "x-dead-letter-exchange": "immediate"
          , "x-message-ttl": 5000  // 5 seconds
          , "x-expires": 10000     // 10 seconds
          , "x-dead-letter-key": "right-now-queue"
        }
      }, function() {
          console.log('publish');
          conn.publish(key, {
            v: count++
          }, {
            contentType: 'application/json'
          });
      });
    }, PUBLISH_RATE);
  });

});
{% endhighlight %}

#### The Consumer (consumer.js)

{% highlight javascript %}
var amqp = require('amqp')
  , conn = amqp.createConnection()
  ;

conn.on('ready', function() {
  console.log('ready');
  conn.queue('right.now.queue', {
      autoDelete: false
    , durable: true
  }, function(q) {
    q.bind('immediate', '#');

    q.subscribe(function (msg, headers, deliveryInfo) {
      console.log(msg);
      console.log(headers);
    });
  });
});
{% endhighlight %}

### Running

You'll need node-amqp installed and you'll need a rabbit server running on localhost with a default configuration.

In one terminall session run:

{% highlight bash %}
node consumer.js
{% endhighlight %}

This will start listening for messages ready to be run and print them out to the screen.

In another terminal run:

{% highlight bash %}
node producer.js
{% endhighlight %}

This will publish a message every 1 second, but they won't come out of the consumer end until 5 seconds after they were published.

### Altering This Code

You don't have to do this exactly as I have. For instance, I am using a topic exchange but I think this should work with any kind of exchange. You also could share all the send later messages in a single queue that never expires. It really depends on your use case how you wish to configure it.

