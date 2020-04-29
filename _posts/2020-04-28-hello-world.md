---
layout: post
title:  "Hello, World!"
date:   2020-04-28 15:19:16 -0700
categories: hello intro jekyll github
---
["Hello, World!"][hello-world-wikipedia] is the traditional "first step" that software engineers take when doing something, whether writing code, testing a framework, or as in my case, setting up a new service or system. It's a quick and dirty way to make sure that something is minimally set up so that further work can commence, often a "sanity check" to make sure we've got the basics set up correctly.

Here's "Hello, World!" a few different ways.

## Java

{% highlight java %}
public class HelloWorld {
    public static void main(String[] args) {
	System.out.println("Hello, World!");
    }
}
{% endhighlight %}

{% highlight bash %}
$ javac HelloWorld.java
$ java HelloWorld
Hello, World!
{% endhighlight %}

## Python

{% highlight python %}
if __name__ == '__main__':
    print('Hello, World!')
{% endhighlight %}

{% highlight bash %}
$ python hello.py
Hello, World!
{% endhighlight %}

## Perl

{% highlight perl %}
#!/usr/bin/perl
print "Hello, World!\n"
{% endhighlight %}

{% highlight bash %}
$ perl hello.pl
Hello, World!
{% endhighlight %}

## Golang

{% highlight go %}
package main
import "fmt"
func main() {
    fmt.Printf("Hello, World!\n")
}
{% endhighlight %}

{% highlight bash %}
$ go build hello.go
$ ./hello
Hello, World!
{% endhighlight %}

## Shell (really?)

{% highlight bash %}
$ /bin/echo 'Hello, World!'
Hello, World!
{% endhighlight %}

None of this, of course, is new or novel. The above lines have probably been written thousands of times (in which case we should refactor them all out and replace with some function call to the canonical version). And of course there are thousands of other languages, frameworks, and platforms where we can do this. Hopefully in future posts I'll jot down things that are at least slightly more useful or interesting for you, dear reader. Or else this will just be a repository of things I want to remind myself about, which is fine too.

[hello-world-wikipedia]: https://en.wikipedia.org/wiki/%22Hello,_World!%22_program
