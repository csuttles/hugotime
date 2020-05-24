+++
author = "Chris Suttles"
categories = ["Python", "Udemy"]
date = 2017-10-15T07:17:10Z
description = ""
draft = false
cover = "/images/2017/10/Screen-Shot-2017-10-14-at-5.16.25-PM.png"
slug = "python-flask-and-mongo-part-2"
tags = ["Python", "Udemy"]
title = "Python, Flask, and Mongo, Part 2"

+++


In my [last post](http://blog.highspeedlogic.org/python-flask-and-mongo/) I talked about a [Udemy course on building webapps with Python, Flask and Mongo](https://www.udemy.com/the-complete-python-web-course-learn-by-building-8-apps). I wasn't quite finished with the last app in the course (Pricealert) at the time of my last post, but the bones were in place. This post is about that long 10% of a project that starts when you are "%90 done".

### Pricealert

I've [committed a lot of updates in the last week](https://github.com/csuttles/udemy-python-webapps/compare/dd76345fcb9f02602177602edeb798cb1df844ed...master), and now the application really looks like a web application and is ready to get broken out into a separate repository. Here are some of the highlights of those changes.

#### Client side password hashing

In the course, early lessons for this app talk about client side password hashing, and how plaintext passwords used in the first two apps are bad. There's a brief explanation of the difference between hashing and encryption, and the plan is to hash passwords client side, then encrypt the hash to store in the database. For logging in, we would then hash the provided password, and [use passlib's pbkdf2_sha512.verify() to compare the hash to the stored value](https://github.com/csuttles/udemy-python-webapps/blob/master/pricealert/src/common/utils.py#L18-L28).

Unfortunately, in later lessons, the course author says that HTTPS is good enough and therefore we don't need to hash client side, and can just pass plaintext passwords since HTTPS/TLS will make things safe. After this, he makes a few changes to back out the underlying support for hashed passwords from the client. 

While I do agree that HTTPS should be used whenever possible, when it comes to security, a layered approach is best. It's easy to get security wrong, so it's a good idea to use HTTPS (with TLS and the strongest ciphers your clients can support), hash passwords in transit, use firewalls at the edge and core of your network, use IDS/IPS, audit access, and use the principal of least privilege everywhere. If you plan for security in multiple layers, you'll be less vulnerable when an exploit is in the wild, or someone configures something wrong.

I ended up using the CryptoJS library for client side hashing, which can be found [here](https://code.google.com/archive/p/crypto-js/). It's pretty old, and in the longer term, it might be better to move to a more active project, like [Forge](https://github.com/digitalbazaar/forge), but it's still better than no hashing at all. I grabbed [just sha512 from the distribution](https://github.com/csuttles/udemy-python-webapps/blob/master/pricealert/src/static/js/sha512.js), and [added a simple function](https://github.com/csuttles/udemy-python-webapps/blob/master/pricealert/src/templates/html_dependencies.html#L7-L17) to generate the hashes in the [login](https://github.com/csuttles/udemy-python-webapps/blob/master/pricealert/src/templates/users/login.html#L6) and [register](https://github.com/csuttles/udemy-python-webapps/blob/master/pricealert/src/templates/users/register.html#L6) forms:

{{< highlight markdown >}}

<script type="text/javascript">function myOnSubmit(aForm) {
    //Getting the password objects
    var inputPassword = aForm['hashed'];
    //Hashing the values before submitting
    inputPassword.value = CryptoJS.SHA512(inputPassword.value);
    //Submitting
    return true;
}</script>

{{< / highlight >}}

While this is very basic web app hygiene stuff, it's not what I usually do, and it's different from the course material. There's still a lot of work to be done on this app in terms of making it more secure, and I'm looking at using [WTForms](https://wtforms.readthedocs.io/en/latest/validators.html) and [Flask Inputs](https://pythonhosted.org/Flask-Inputs/) for validation, since input validation is glaring omission from the current state. It's important to sanitize and validate user input:

![Little Bobby Tables](https://imgs.xkcd.com/comics/exploits_of_a_mom.png)


### Bootstrap

The course was recorded using bootstrap 3.x, but I prefer to stay close to the current release of things when I can, and [Bootstrap 4.x is in beta after a *long* series of alpha releases](https://getbootstrap.com/docs/4.0/getting-started/introduction/). There's a lot of changes between these major versions, because major versions are where you break backward compatibility; here's a [good summary of those changes](https://www.quackit.com/bootstrap/bootstrap_4/differences_between_bootstrap_3_and_bootstrap_4.cfm).

This meant that at least 50% of the classes used for styling in the course were deprecated and I had to spend some time in the docs translating equivalent classes. Fortunately, [the documentation for Bootstrap](https://getbootstrap.com/docs/4.0/getting-started/introduction/) is outstanding. I think that I got more out of the course as a result of my efforts to use v4 instead, and while I don't intend to become a front-end person anytime soon, I feel comfortable enough with bootstrap that I found myself translating things in my inner narrative while watching the course once it neared completion. I know just enough Bootstrap to make things look decent, and that means good support across browsers and on mobile. That's enough for me. :)

### Glyphs

The course also used Bootstrap 3 glyphicons, which are no longer part of Bootstrap, so instead I used [Github's Octicons](https://octicons.github.com/). There's a great selection of icons, and boilerplate code for many frameworks. While there weren't examples for Bootstrap, they were still easy to use by simply embedding in an `<img>` tag.

### Fixing Item Price Parsing

Aside from all the window dressing, I found a problem with the parsing in the `Item.load_price()` [method](https://github.com/csuttles/udemy-python-webapps/blob/master/pricealert/src/models/items/item.py#L27-L43). The small, but important change is visible in the commented assignment of `string_price` below:

{{< highlight markdown >}}

    def load_price(self):
        # https://www.redbubble.com/people/immortalloom/works/22929408-official-big-o-cheat-sheet-poster?p=poster&finish=semi_gloss&size=large
        #         <meta itemprop="price" content="32.66"/>
        # tag_name = meta
        # query = { "itemprop": "price" }
        request = requests.get(self.url)
        content = request.content
        soup = BeautifulSoup(content, 'html.parser')
        element = soup.find(self.tag_name, self.query)
        #string_price = element.text.strip()
        string_price = str(element)

        pattern = re.compile(r'(\d+\.\d+)')

        match = re.search(pattern, string_price)
        self.price = float(match.group())
        return self.price
 
{{< / highlight >}}
