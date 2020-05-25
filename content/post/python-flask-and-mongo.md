+++
author = "Chris Suttles"
categories = ["Python", "Udemy"]
date = 2017-10-07T22:59:07Z
description = ""
draft = false
cover = "/images/2017/10/big-o-cheatsheet.jpg"
slug = "python-flask-and-mongo"
tags = ["Python", "Udemy"]
title = "Python, Flask, and Mongo"

+++


I decided to take a [Udemy course on building webapps with Python, Flask and Mongo](https://www.udemy.com/the-complete-python-web-course-learn-by-building-8-apps), and it's been a lot of fun. You can find my Github repo for the course here: [https://github.com/csuttles/udemy-python-webapps](https://github.com/csuttles/udemy-python-webapps).

I'm not a web developer. I've always been focused on writing backend stuff, and operational glue. That's part of what made me decide to do this. I wanted to spend some time just playing with Python and playing means doing something a little different than what I do at work.

In retrospect, I should have broken each of the projects into their own repos. There are some other things I would have done differently too, but hey, hindsight is a gift you get after the fact. The first few apps built in this course were so trivial they are not mentioned here, and that started the pattern of dumping everything into a single repo.

#### Termblog

The first app I decided to actually commit is a *very simple* terminal blog app. It's the foundation for the course and walks through setting up Pycharm, building a project, a virtualenv and introduces the pymongo and flask libraries. The user credentials are stored in plaintext, and for anything other than practice, this app is pretty useless and awful. It does the job of introducing the course and tools, however.

#### Webblog

With this project, things get more interesting. After the remedial/level set in the termblog project, this project builds on those concepts and makes things a little more complex. Credentials are still stored in plaintext (cringe), but this goes a lot farther with flask and some important concepts are briskly introduced. This project lacks a single try/except clause, but introduces jinja templates and also the [Bootstrap framework](https://getbootstrap.com/), which I really liked. I'm not terribly interested in tweaking design (remember that part about not being a web developer?), so using bootstrap was a pretty easy and mostly painless way to get a much better looking skin on the project after building the core in Python.

#### Pricealert

This is the final project for the course, and is more complex than both of the preceeding projects. It builds on all the things covered in the course, including some callbacks to projects I skipped here due to simplicity.  Passwords are finally managed using hashing and stored using encryption. The premise of the application is a little contrived; the idea is that users can set up alerts for a price drop of specific items on a website and be notified when said item drops below a user specified threshold. The concept of flask blueprints is introduced, and the project is structured in a more realistic way. The price checking is handled via requests and beautifulsoup, and a whirlwind introduction to regex. I was able to follow all of this really easily, since I have used all these tools before. I think someone with less experience would find the course more challenging or be overwhelmed. All of the projects follow an OOP style, and in this project there's finally try/except clauses for things. There's also a super brief introduction to class inheritance via an Exception subclass, with additional subclasses for raising and handling within the project. This project introduces some more interesting operations with pymongo, including the update/upsert, searching and comparisions. It also introduces the Python debugger in [PyCharm](https://www.jetbrains.com/pycharm/), walking through settings breakpoints and stepping into and over code, along with using both the Python console in PyCharm, and [Postman](https://www.getpostman.com/apps) for verifying and debugging while developing the API.


There's an atrocious method implementation for searching mongo for a matching string prefix, which queries the db repeatedly adding a single character each time until only a single result is returned. Since the scheme is stored as part of each URL, this means that the first 7-8 times this query occurs, there's a bunch of iterations that are just wasted.

Here's the example code:

{{< highlight markdown >}}
@classmethod
def find_by_url(cls, url):
    for i in range(0, len(url) + 1):
        try:
            store = cls.get_by_url_prefix(url[:i])
        except:
            raise StoreErrors.StorenotFoundException(
                "The URL prefix used to find the store didn't give us any results.")
        return store
{{< / highlight >}}


Here's some small changes I made to make it less awful:

{{< highlight markdown >}}
@classmethod
def find_by_url(cls, url):
    """

    :param url: item url
    :return: store or raise exception
    """
    # example was O(n) on an expensive database call
    # made the range more appropriate, skip scheme and better cadence
    # It's a cheap efficiency increase, but still could be better
    for i in range(10, len(url) + 1, 3):
        try:
            store = cls.get_by_url_prefix(url[:i])
        except:
            raise StoreErrors.StorenotFoundException(
                "The URL prefix used to find the store didn't give us any results.")
        return store
{{< / highlight >}}

For completeness/convenience sake, here's the get_by_url_prefix method being called:

{{< highlight markdown >}}
@classmethod
def get_by_url_prefix(cls, url_prefix):
    """
    :param url_prefix: url_prefix to match on in mondogdb
    :return: return store that startswith url_prefix: url_prefix
    """
    return cls(**Database.find_one(StoreConstants.COLLECTION, {"url_prefix": {"$regex": '^{}'.format(url_prefix)}}))
{{< / highlight >}}

I'm not quite done with this application, but the core of the Python work is complete. I tested an alert using test data from [this page](https://www.redbubble.com/people/immortalloom/works/22929408-official-big-o-cheat-sheet-poster?p=poster&finish=semi_gloss&size=large) on Redbubble, which is a [Big-O cheatsheet](http://bigocheatsheet.com/) poster. The example test data in the course points to an item that is no longer sold and therefore doesn't work. There's also a big chunk of the course that discusses parsing using amazon as an example, and ends with something like "this won't work because Amazon is too smart and you have to use their API". 

I ended up populating the database with my own user, item, store and alert data, and then ran a script that uses the libs authored in the course to do the check. That script looked at the last time the alert was updated in the database, compared it to a threshold for updates, then pulled the store from the database to get the parameters to pass to beautifulsoup during parsing, requested the URL, parsed the response, got the current price, which was lower than the alert price, and finally [sent me an email via mailgun](https://www.mailgun.com/) alerting me the program found a deal for me, including a link to my item. The last part of the course is a deeper dive into bootstrap and jinja template work to make the web interface, and then deploying it via Heroku. I'm probably going to set up deployment using [AWS Codestar instead (got those free credits!)](http://blog.highspeedlogic.org/aws-codestar-challenge/), but since the core of the work in Python is done, and I am very close to the end of a course I really enjoyed, I wanted to share my experience here. Udemy runs sales all the time, so I think I picked this up for $10-15, and it was fun and worth it.

