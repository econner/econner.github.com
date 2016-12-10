---
layout: post
title: Understanding Evented Servers
category: writing
---

# {{page.title}}

<span class="meta">January 13 2013</span>


I've been using and trying to understand evented servers in python for
a little while now.  In this post I'd like to give an overview of what it
means to have an evented server and show an example implementation.

I'd first like to start with some basics that I realized were gaps in my
understanding of how things work that set me back:

* What exactly is a connection to a server?
* What is a socket?

## The Connection

Your computer uses TCP when browsing the web to transmit data back and forth
between you (the client) and the server.  TCP uses connections.  Now, it is
incorrect to think of a TCP connection as some physical, dedicated wire going from
google.com to your computer.

Obviously this isn't how the internet works, but there's a tendency
to think of the abstraction in that way.  In reality a connection is nothing more
than a 4-tuple: (source ip, source port, destination ip, destination port).
This 4-tuple also uniquely defines a socket.  Any socket with a field differing
in any one of these values is a different socket.

The process of opening a connection is the exchange of this information so that
the client knows where to reach the server and the server knows where to send
data back to the client.

## TCP Sockets

The TCP socket then is an abstraction provided by the OS to achieve a reliable,
in-order byte stream on both sides of the connection.  TCP layers on mechanisms for
re-ordering packets that arrive out of order and re-trying sent packets that
get lost in the network.

In the traditional TCP server model, a new thread or process is initiated per
connection.  Let's say, for example, we have a site http://example.com/ with
ip address 55.55.55.55.  Now two clients establish connections to our server:

	(123.123.123.123, 50000, 55.55.55.55, 80)
	(124.124.124.124, 50000, 55.55.55.55, 80)

Each connection has the same destination IP and destination port since they
are connecting to the same website.  Each of these connections however defines
a different socket since they have different source addresses.  A server such
as Apache will spawn a new thread or process to serve each of these connections.
The OS will provide a separate interface to access each of these sockets, and
each socket will use a separate file descriptor.

## Blocking TCP Server in Python

The following simple server example taken from
[Scot Doyle's epoll Tutorial](http://scotdoyle.com/python-epoll-howto.html)
does not use threading, but illustrates how a server accepts connections
and where it could spawn off a new process to handle each connection.

{% highlight python %}
import socket

EOL1 = b'\n\n'
EOL2 = b'\n\r\n'
response  = b'HTTP/1.0 200 OK\r\nDate: Mon, 1 Jan 1996 01:01:01 GMT\r\n'
response += b'Content-Type: text/plain\r\nContent-Length: 13\r\n\r\n'
response += b'Hello, world!'

serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
serversocket.bind(('0.0.0.0', 8080))
serversocket.listen(1)

try:
   while True:
      connectiontoclient, address = serversocket.accept()
      request = b''
      while EOL1 not in request and EOL2 not in request:
          request += connectiontoclient.recv(1024)
      print('-'*40 + '\n' + request.decode()[:-2])
      connectiontoclient.send(response)
      connectiontoclient.close()
finally:
   serversocket.close()

{% endhighlight %}

First, comment out the following line as shown:

{% highlight python %}
# connectiontoclient.close()
{% endhighlight %}

Start the server and connect to it in a browser.  Then from a shell run:

{% highlight sh %}
$ netstat -anl
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address     Foreign Address     (state)
tcp4       0      0  127.0.0.1.8080    127.0.0.1.52356    ESTABLISHED
tcp4       0      0  127.0.0.1.52356   127.0.0.1.8080     ESTABLISHED
tcp4       0      0  *.8080            *.*                LISTEN
{% endhighlight %}

This shows the established TCP connection from the client to the server and
vice versa (since in this case both are on the same machine).

The above example accepts connections on port 8080 for ip address 0.0.0.0 (which just
means all ip addresses available to the machine).  Accepting a connection
returns a new socket object ``connectionclient`` that is usable to send and
receive data on the connection.

The new socket utilizes a separate file descriptor from ``serversocket`` so a new
thread could be spawned here with access to the new socket in order to
process the client's request while other requests could be served by spawning
threads in the parent process.

## Non-blocking Sockets

Many newer servers use non-blocking sockets in order to serve many more requests
than ever before from a single thread.  The idea of a non-blocking socket is one
that, instead of blocking the calling thread, notifies the OS of read / write readiness
via events.  They typically use an event notification mechanism in Linux called
epoll.

The typical way then that a server uses epoll is to create an epoll object,
specifying which events to monitor for specific sockets.  Then the server loops infinitely
asking the epoll object for a list of events that have occurred since the last
query.  Based on which readiness events occur, actions are performed on sockets that
are ready for those events and the epoll object is modified to listen for
new events accordingly.
