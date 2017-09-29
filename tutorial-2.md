Database-backed Flask development on PythonAnywhere: database migrations and virtualenvs

Welcome to the second part in our tutorial for getting started with Flask development on PythonAnywhere!
This is a continuation of [our previous tutorial]((https://blog.pythonanywhere.com/121/)), so if you haven't
been through it, you should do that first.  In particular, this tutorial will assume that you're starting
with the code that you had at the end of the last one: a simple website that allowed you to post comments on
a page, all of those comments being stored in a database:

<img width="500" src="/static/images/flask-tutorial-final-result.png">

It was made up of two files; a Python one which controlled what it did, and a template one that controlled
how it was displayed.  All of our code was under source-code control, and it was running as a website on
PythonAnywhere.

Obviously this was a rather limited website!  It would be nice if we could add some features -- and in this
tutorial, that's exactly what we'll do.   Our original site used the slightly ugly password protection
built in to PythonAnywhere; this means that in order for people to post on it, they had to enter a username
and password -- but it was the same username and password for everyone, which isn't terribly secure.  So we're
going to add our own login page -- which means that we can also store a record of who posted what, and when.

Here's what it will look like:

IMAGE HERE -- login page

IMAGE HERE -- main page

Adding these new features will require us to do a bit of work in the background; our existing site works
fine for what it is, but it's not very extensible, so we'll fix that as we go.   Let's get started!

## Handling login and logout

Just like we did before, we'll start off by designing how the site should work.   We already have a
page that lists all of the comments, and allows people to add new comments.   But we don't have a login page,
so let's write one.   Log in to PythonAnywhere, then go to the "Files" tab, and then down into the "mysite"
directory that contains your code, and then to the "templates" subdirectory.   Create a file called "login_page.html"
with the following content:

    <html>
        <head>
            <meta charset="utf-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1">

            <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css" integrity="sha512-dTfge/zgoMYpP7QbHy4gWMEGsbsdZeCXz7irItjcC3sPUFtf0kuFbDz/ixG7ArTxmDjLXDmezHubeNikyKGVyQ==" crossorigin="anonymous">

            <title>My scratchboard page: log in</title>
        </head>

        <body>
            <nav class="navbar navbar-inverse">
              <div class="container">
                <div class="navbar-header">
                  <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
                    <span class="sr-only">Toggle navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                  </button>
                  <a class="navbar-brand" href="#">My scratchpad: login page</a>
                </div>
              </div>
            </nav>

            <div class="container">

                <div class="row">
                    <form action="." method="POST">
                      <div class="form-group">
                        <label for="username">Username:</label>
                        <input class="form-control" id="username" name="username" placeholder="Enter your username">
                      </div>
                      <div class="form-group">
                        <label for="password">Password</label>
                        <input type="password" class="form-control" id="password" name="password" placeholder="Enter your password">
                      </div>
                      <button type="submit" class="btn btn-success">Log in</button>
                    </form>
                </div>

            </div>

        </body>
    </html>

This should be pretty clear -- most of it is boilerplate, and the form inside the row (inside the container inside the body)
is a simple login form with username and password fields.   Note that the `type` of the password field is "password" --
this is to do the normal trick of making it display asterisks when people type into it.

So, we have an HTML template that will display our login page.   Let's add the Flask code to use it!   Keep the
browser tab showing the template file open -- we'll come back to it in a bit -- and in a new browser tab, load up the
`flask_app.py` file, and add a new view like this:

    @app.route("/login/")
    def login():
        return render_template("login_page.html")

Reload the app from the "Web" tab or by using the button at the top right of the page, then go to your site.
You'll get the page with the list of comments, of course -- so next, edit the URL in the browser's address bar
and add "/login/" to the end.   Hit return, and we get the login page.   So far, so good!  But if you
enter a username and a password into the form, you'll get a "Method not allowed" error.  Hopefully that will be
familiar from the first tutorial; our form is sending a `POST` request when the "Log in" button is clicked, and our
view doesn't handle that yet.

We've reached a state where our site is doing something new, so let's use git to take a checkpoint.   In a new
browser tab, open up a Bash console from the PythonAnywhere "Consoles" tab.   Change your working directory
to `mysite`, which is where all of our code is, by running this command:

    cd mysite

...then run "git status" to see the changes:

IMAGE HERE

We have a new template file, which we need to add to the git repository:

    git add templates/login_page.html

...and some changes to `flask_app.py`:

    git add flask_app.py

And now we've added the files, we commit those changes to make a coherent single change:

    git commit -m"First cut at the login page"

Right, let's add the basics of the login functionality.   Initially, we won't actually distinguish between logged-in
and non-logged-in users on the main page; we'll just change this page so that if someone enters the username "admin" and the password
"secret", they get sent back to the front page -- but if they enter anything else, they'll get sent back to the login
page.   Change the view so that it looks like this:

    @app.route("/login/", methods=["GET", "POST"])
    def login():
        if request.method == "GET":
            return render_template("login_page.html", error=False)

        if request.form["username"] != "admin" or request.form["password"] != "secret":
            return render_template("login_page.html", error=True)

        return redirect(url_for('index'))

Let's break that down:

        if request.method == "GET":
            return render_template("login_page.html", error=False)

If the browser is just visiting the page normally, we render the login page passing a variable called `error` in,
setting that variable to False.

        if request.form["username"] != "admin" or request.form["password"] != "secret":
            return render_template("login_page.html", error=True)

Because we got past the first step in the function, we now know that this request is the result of someone
trying to log in.   If either the username they specified isn't "admin", or the password isn't "secret",
we render the login page again, but this time we set the `error` variable to True.

        return redirect(url_for('index'))

Otherwise, we just send them back to the index page -- the main page of the app.

Before we try this out, head back to the tab you kept open with the template in it, and add this to the login page,
just before the form:

    {% if error %}
        <div class="alert alert-warning" role="alert">
            Incorrect username or password
        </div>
    {% endif %}

Reload the website, then visit the login page again.  Try entering an incorrect username and password, and click
the "Log in" button -- you'll get a yellow warning at the top of the page.   Now try logging in with the username
"admin" and the password "secret": you'll be taken to the main front page.

That's actually quite a big step forward: now we have a login page that is checking credentials, which gives us
the beginnings of some security.   Let's get that committed: go to the tab where you have the bash console running,
and use "git add" and "git commit".

So, the next step is to actually change the behaviour of the site based on whether the user is logged in
or not.   For this, we're going to use an extension to the Flask web framework called
"<a href="https://flask-login.readthedocs.io/en/latest/">flask-login</a>".  Like "flask-sqlalchemy", which
you'll remember we're using to talk to the database, "flask-login" is not part of Flask itself, but provides
useful functionality that many sites find useful.  Conveniently enough, it is
already installed on PythonAnywhere, so we can just use it without having to install anything.

Go to the browser tab where you have the `flask_app.py` file open, and add an import line like this next to the
one for flask-sqlalchemy:

    from flask_login import login_user, LoginManager, UserMixin

Next, just after the line where we set up the database, which looks like this:

    db = SQLAlchemy(app)

...add three lines to set up the login system:

    app.secret_key = "something only you know"
    login_manager = LoginManager()
    login_manager.init_app(app)

The first of those lines is a placeholder -- you should replace the words "something only you know" with a
reasonably long (say, 20 characters) completely random string (letting a cat walk across the keyboard works
well, but if you can't find a nearby cat, just hammer randomly on some keys).   This is because Flask-Login is going to use
some cryptographic components of Flask, and those need to have a secret key to operate.   The next two lines
just create an instance of the Flask-Login system and associate it with your Flask app.

So now we've imported and are using the login system; the next step is to tell it about some users.
The first thing is to define a class to describe what users look like; we'll have an instance of this
for each user.   Add it just before the `Comment` class.

    class User(UserMixin):

        def __init__(self, username, password):
            self.username = username
            self.password_hash = generate_password_hash(password)


        def check_password(self, password):
            return check_password_hash(self.password_hash, password)


        def get_id(self):
            return self.username

This class is a subclass of `UserMixin`, which is provided by Flask-Login.  This is because the extension is
very flexible, and allows you to set things up in all kinds of weird and interesting ways -- but if all you want
is the same kind of concept of a "user" that most websites have, it provides the `UserMixin` class to inherit
sensible behaviour from.  We specify that our users have usernames and passwords, and that they have a unique
identifier that is simply the username.

There's something that's well worth drawing your attention to in that code; the "password_hash" stuff.  Storing
passwords in plain text is a really bad idea; if a hacker gets in to your system, they get a list of usernames
and passwords.  If those username/password combinations are used on other sites, the users are in trouble.

So what we do is "hash" and "salt" the passwords, which means that the passwords themselves aren't going to be stored in
the `User` objects -- instead, a cryptographic string that pretty-much uniquely identifies the password will be.
For more details, there's
[an excellent article on hashing and salting in the Guardian](https://www.theguardian.com/technology/2016/dec/15/passwords-hacking-hashing-salting-sha-2)

Having added that, we need to import the two hash functions that we're using:

    from werkzeug.security import check_password_hash, generate_password_hash

Now, let's specify a couple of users.  Normally
a website would keep its list of users in the database -- and we'll change things to do that later -- but for
now, we'll just keep them in our code.  Add this just after the `User` class:

    all_users = {
        "admin": User("admin", "secret"),
        "bob": User("bob", "less-secret"),
        "caroline": User("caroline", "completely-secret"),
    }

(Of course, having done all of that clever stuff so that we don't have the passwords in plaintext, we're
now adding code with... the passwords in plaintext.  Don't worry -- that's going to change by the end of this
tutorial!)

So that's a dictonary that maps from the username to the user object for three users.   Why a dictionary rather
than a simple list?   It's because Flask-Login needs to be provided with a function that, given a unique ID
(as returned by the `get_id` method on a `User` object) returns the associated `User` object.  Let's add that,
just after the dictionary:

    @login_manager.user_loader
    def load_user(user_id):
        return all_users.get(user_id)

Now we've set things up so that Flask-Login is properly configured, so now let's use it.   Change the `login`
view so that it looks like this:

    @app.route("/login/", methods=["GET", "POST"])
    def login():
        if request.method == "GET":
            return render_template("login_page.html", error=False)

        username = request.form["username"]
        if username not in all_users:
            return render_template("login_page.html", error=True)

        user = all_users[username]
        if not user.check_password(request.form["password"]):
            return render_template("login_page.html", error=True)

        login_user(user)
        return redirect(url_for('index'))

So now we check whether a login attempt is for a known user, and if it isn't, we dump them back on the login page
with an error.  If it is a real user, we check the password, and if it's wrong, we also error out.   Finally,
if it's a real user, we call Flask-Login's `login_user` function to stash away the fact that they're successfully
logged in.

Before we fully make use of this, let's check whether it works at least well enough to check things against our
current list of users.   Reload the site using the button to the top right of the editor, then visit the login
URL again.   Enter an invalid username/password; you'll get a login error.   Now try logging in as user "bob" with
password "less-secret" -- it'll take you to the front page.

So everything is working reasonably well -- time to "git add" and "git commit" just so that we have something
to go back to if we break stuff.

Our next step is to actually use this login stuff to protect our site.  First, load up the template for the main
page.   Near the bottom, we have the HTML that defines the input field where people can add comments:

    <div class="row">
        <form action="." method="POST">
            <textarea name="contents" placeholder="Enter a comment" class="form-control"></textarea>
            <input type="submit" value="Post comment">
        </form>
    </div>

What we want to do is hide that if the person viewing is not logged in.   This is really easy to do; Flask-Login
makes a variable called `current_user` available in our templates.  If a user is logged in, the field
`is_authenticated` on that variable is set to False.   So, we can do this:

    {% if current_user.is_authenticated %}
        <div class="row">
            <form action="." method="POST">
                <textarea name="contents" placeholder="Enter a comment" class="form-control"></textarea>
                <input type="submit" value="Post comment">
            </form>
        </div>
    {% endif %}

Save that change, reload the site, and go to the main page, and...

Ah.  We're already logged in, after our test of the login page a few minutes ago -- so we still see
the form to enter a comment.   Not a great test!  We need to somehow log out.  So let's write the
view for that.

*(An aside: some people might have noticed that the fact that we're logged in has persisted even though
the website has been reloaded -- that's different to what happened in the first tutorial, when we had to
store comments in a database before things would persist.   This is because Flask-Login is doing clever
stuff with browser cookies.)*

Go to the browser tab where you have `flask_app.py`, and add a new view:

    @app.route("/logout/")
    @login_required
    def logout():
        logout_user()
        return redirect(url_for('index'))

...and add `logout_user` and `login_required` to the list of things that we're importing from `flask_login` near the top of the
file.  Note the new decorator on the view, `login_required` -- this is provided by Flask-Login and allows you
to protect views so that they can only be accessed by logged-in users.

It would be nice to provide an easy way to log in and log out from the front page of the site, so go back
to that template, and change the `nav` section at the top so that it has a "log in" link if the user is not logged
in, or a "log out" link if they are.   Here's the complete `nav` section that you need, but the only new stuff
is the `ul` section near the end, the rest should already be there:

    <nav class="navbar navbar-inverse">
      <div class="container">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#">My scratchpad</a>
        </div>
        <ul class="nav navbar-nav navbar-right">
            {% if current_user.is_authenticated %}
                <li><a href="{{ url_for('logout') }}">Log out</a></li>
            {% else %}
                <li><a href="{{ url_for('login') }}">Log in</a></li>
            {% endif %}
        </uk>
      </div>
    </nav>

Once you've done that, reload the website, and then go to your tab showing the site.   Hit the browser
refresh button, and you should see a "Log out" link near the top right:

IMAGE HERE

Click that, and you'll be taken to the front page again -- but the place to enter a comment will have
disappeared!

IMAGE HERE

Click the "Log in" link, and you'll be taken to the login page, where you can log in as admin, bob or caroline,
using the passwords we defined in the code.   When you do that, you'll be taken to the main page, and you'll be able
to add comments again.

That's definitely worthy of a commit!  We have a secure-looking site.  Head over to the bash console, `git add`,
`git commit`, you know the drill by now :-)

Now, you might have noticed that I said "a secure-*looking* site" just then.  It does look secure, but it actually
isn't.  Although we don't display anything on the web page to allow someone to add a comment to your site, there's
nothing stopping people from sending carefully-crafted requests that look as if they're coming from the web
page down to your Flask server and adding comments.  You can get a feel for the problem by:

* Going to the front page, and logging in (if you're not already logged in)
* Right-click on the "Log out" link, and open it in a new tab.  You're now logged out.
* Type a comment into the still-visible "enter a comment" field, and submit it.

You'll see that the comment was added, even though you were logged out.  A hacker who wanted to put stuff on your
page wouldn't need even be logged in in the first place; they could use a tool like `cURL` to send POST requests
directly to the Flask server -- without even using a browser.   (If you're interested in how hackers do stuff like
this, Rob Graham of Errata Security wrote
[a 'Hacking for beginners' kind of blog post](http://blog.erratasec.com/2017/09/browser-hacking-for-280-character-tweets.html)
explaining some of the details, in the context of getting early access to Twitter's 280-character tweets.)

What we need to do is add some server-side validation -- that is, when we receive a POST request that is trying
to add a comment to our database, we should check first whether or not the user is logged in.  This is a simple
change to the `index` view in `flask_app.py` -- just add this, just before the line that creates a new `Comment`
object:

    if not current_user.is_authenticated:
        return redirect(url_for('index'))

...and also add `current_user` to the set of things we import from `flask_login` at the top of the file.  Reload
the website, and try going through those three steps that showed the problem before.  This time, when you try
to post a comment as a non-logged-in user, it'll just take you back to the main page without storing the comment.
Phew!

Now we have a site that actually is secure :-)  Commit those changes, and -- if you like -- switch off the password
protection at the bottom of the "Web" tab inside PythonAnywhere -- we have something better now, so we don't need it.


## Adding timestamps

So, we can log in and log out.   Our goal for this tutorial is to be able to say who posted comments, and when.
The timestamp is the easiest one of these, so let's add that first!

As always, we start off by designing how it will look; initially pass the current time to the template
and rendering that, along with the "name" anonymous, for every comment -- just so we can get a feel for what it will look
like.   To do that, add these lines to the top of the file to import the `datetime` class from the Python standard library:

    from datetime import datetime

...then change the code that renders the template in `index` from this:

    return render_template("main_page.html", comments=Comment.query.all())

...to this:

    return render_template("main_page.html", comments=Comment.query.all(), timestamp=datetime.now())

Now, reload the web app and take a look at the site -- it won't have changed.  This is a good thing!

Next, in a new browser tab, open the template file.  Right now, we have a `for` loop and inside it, for each comment, we have this HTML:

                <div class="row">
                    {{ comment.content }}
                </div>

Replace it with this code:

                <div class="row" style="margin-bottom: 1ex">
                    <div>
                        {{ comment.content }}
                    </div>
                    <div>
                        <small>Posted {{ timestamp.strftime("%A, %d %B %Y at %H:%M") }} by anonymous</small>
                    </div>
                </div>

Check out the site again -- now you'll see that each comment has a timestamp next to it, saying something
like "Posted Friday, 01 September 2017 at 17:44 by anonymous".   Because we're not storing the timestamps yet, all of
them will have the same timestamp -- the time at which the page was rendered.  But it's a start!

If you prefer US-style date formatting, where the month comes before the day, the change is really simple --
just swap around the `%d` and the `%B` in the template where it says

    {{ timestamp.strftime("%A, %d %B %Y at %H:%M") }}

What that line means is, call the `strftime` method on the `timestamp` object that we're passing in to the
template.  You can read more about what all of the different options for the `%` bits in there mean on
[this Python documentation page](https://docs.python.org/3/library/time.html#time.strftime).

Once you have everything looking the way you want, let's take a checkpoint.  Just to keep things interesting,
do a "git diff" to see the changes -- then the normal "git add" / "git commit".

## Virtualenvs

Now it's time to write the code to actually save timestamps -- we'll do commenter names later.  This is a
little tricky.  Right now, we have a number of comments on our site.  Those comments don't have timestamps,
and even worse, they're stored in a database table that doesn't have a column where we could store timestamps.
Now, of course, for a website we're just building to play with like this one, we could just delete everything
and build the database up from scratch, this time with a timestamp field.   But for a real site that would
be a bit destructive.

What we need to do is change the database in a controlled way.   We want to be able to add a new timestamp
field to our `Comment` class, which is as easy as you'd expect, but we also want to be able to change the
database to add the new column, putting a sane value in there for the existing comments.

It is actually possible to do that by starting a MySQL console and changing things by hand, but it's a lot
of hassle -- and doesn't work very well when you're making regular changes as you extend your site.   So
we're going to use something called *migrations*.   A migration is, conceptually, a change to a database's
structure packaged up as a small Python file that knows how to make the change.

Just like there is Flask-SQLAlchemy to interface with the database, and Flask-Login to handle logins, there's
a Flask extension that knows how to do migrations for us.   One problem, though -- it's not installed on
PythonAnywhere by default.   So we're going to need to install a new module.

The way we're going to do that is to use a thing called a "virtualenv"; there are
[a couple of ways to install packages on PythonAnywhere](https://help.pythonanywhere.com/pages/InstallingNewModules),
but [virtualenvs are the best in the long run](https://help.pythonanywhere.com/pages/VirtualenvsExplained).
They're basically private stores inside your PythonAnywhere account of the specific modules (Flask and its extensions)
that your site depends on -- check out the link above if you want the details of how they work.

In your bash console, and run the following command:

    mkvirtualenv flask-tutorial --python=python3.6

It will take a little time to run.   The command `mkvirtualenv` comes from a helpful package called
`virtualenvwrapper`; virtualenvs can be a little fiddly to work with if you use the normal tools, and
virtualenvwrapper provides some helpful, well, wrappers.  The parameters that we've given it are first,
a name for our virtualenv, `flask-tutorial`, and secondly, a specification of which version of Python
we want to use -- any given computer can have multiple versions installed (PythonAnywhere has a bunch,
unsurprisingly) and a virtualenv normally only supports one, so you need to be clear which one you want to use.

Once the command has completed, you'll notice that the prompt has changed a bit.   Before it would have been
something like this:

    17:39 ~/mysite (master)$

But now it will look like this:

    (flask-tutorial) 17:42 ~/mysite (master)$

When you're in a virtualenv, the prompt shows you its name at the start.  This is very useful if you have
lots of them, of course.

Now, our next step is to install some packages using `pip`.   Initially, we'll just get our virtualenv to contain
the packages we need right not -- not the migration stuff we're about to use.  There are three that we need to install:
Flask itself, the Flask-SQLAlchemy plugin, and mysqlconnector, a library that helps SQLAlchemy talk to MySQL.
They also have various dependencies -- Flask depends on a templating library called Jinja, for example -- but
you don't need to worry about those -- `pip` will automatically install them for you.   So all you need to do
is run this command:

    pip install flask flask-sqlalchemy mysql-connector-python

That will take a little while to run.  It will pull down all of the packages from PyPI, along with their
dependencies, and install them into your virtualenv.   Put your feet up for a bit, or make a cup of tea.

When it's finished and you get the bash prompt back, you can see what it's installed by running the
`pip freeze` command.  You'll see something like this:

    -f /usr/share/pip-wheels
    click==6.7
    Flask==0.12.2
    Flask-SQLAlchemy==2.2
    itsdangerous==0.24
    Jinja2==2.9.6
    MarkupSafe==1.0
    mysql-connector-python==2.0.4
    SQLAlchemy==1.1.11
    Werkzeug==0.12.2

You can see that it's showing you not only the packages you installed, but also the dependencies, and
it's showing the version number for each one.

Just out of interest, let's compare that to the set of packages that you had installed when you weren't inside your
virtualenv.   Run the following command to exit the env:

    deactivate

You'll see that the bash prompt will lose the `(flask-tutorial)` prefix.   Now, let's see what PythonAnywhere
has installed by default for Python 3.6:

    pip3.6 freeze

You'll get several pages-worth.   All very useful when you're just getting started, but we're branching off
into more advanced stuff now and taking control of our dependencies :-)   Let's go back into our nice, pared-down,
minimal virtualenv:

    workon flask-tutorial

So, we have a virtualenv.  But our website is still using the packages that were installed system-wide, so
it's not being used.  The next step is to fix that.  Open a new tab by right-clicking the PythonAnywhere logo,
and go to the "Web" tab.   Make sure that your website is selected, then scroll down a bit.   You'll see a
section with the header "Virtualenv":

IMAGE HERE

Click on the link to edit the field, and enter the name of the virtualenv you created earlier,
`flask-tutorial`.  The system will think for a second, and then fill in the full path to the virtualenv.
A virtualenv is just a directory in your private file storage -- but because you're using virtualenvwrapper, it's in
a specific directory that PythonAnywhere knows about, so you can just specify the name.

Now that we've made a change on the "Web" tab, the next step is to reload the website to make that change
take effect.  Scroll back up to the top of the page, and click the green "Reload" button (which is the stand-alone
equivalent of the reload button in the editor that we were using in the last part of the tutorial).  Wait for the
spinner to disappear, and then right-click your site's name to visit it.  If all went well, it should display
just like it did before, showing all of the comments that were already there.  Hooray!

IMAGE HERE

Well, maybe not quite "hooray", as all we've done is avoid breaking things -- however, sometimes that's just
how coding works ;-)

It's good practice to keep a list of the external packages that your code depends on inside your Git repository.
The tradition is to use a file called `requirements.txt`.   In order to avoid having to type everything in, let's
use a bash command.  Go back to your bash console.

We want to create a file called `requirements.txt` that lists the packages we use.  And we already know
there's a command `pip freeze` that will produce that list.  So we can run the command and tell it to write its
output to a file:

    pip freeze > requirements.txt

If all goes well, that command will just send you back to the prompt.   Now, open another tab by right-clicking
the PythonAnywhere logo, and go to the "Files" tab, then down into the `mysite` directory.  You'll see the file
`requirements.txt` -- click on it to load it into the editor.

What we want to do here is remove the first line, the one that says `-f /usr/share/pip-wheels`; it's not actually
one of our requirements, it's just some extra junk that pip has printed.  So remove that line, then save the file.

Now let's get the requirements file into our repository.  Back in the bash console, if you run `git status` you'll
see that `requirements.txt` shows up in red as an "untracked file" -- remember, git needs to be explicitly told that
it should track new files.   So, use `git add` to add it to the list of tracked files, then `git commit` to take a
checkpoint and add it to the repository:

IMAGE HERE


## Migrations

OK, so now we have a virtualenv and we want to install the Flask database migration extension into it.  Logically enough,
it's called [Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/).  To install it, go to your
bash console, and run this:

    pip install Flask-Migrate

That will take a minute or two to install, as Flask-Migrate depends on a much larger module called `alembic`,
which actually does all the migration stuff.

Once it's all installed, we need to configure our code to use it.  We do this *before* we make any changes to
the database, because we're going to have to run some commands first in order to let the system know what the
current state of the database is before we start adding stuff.   When you want to add a new thing to the database,
you add the thing you want to the Python code, then run a command to create a new migration that will
add it to the database.  The command works out what to do by comparing what it sees in the database with
what the code seems to expect, then generating a migration that changes the database in the appropriate way.
If that all sounds a bit hard to get your head around, don't worry -- it'll become clearer once we've worked
through the example.

In your Python code editor, add a new line just underneath the bit where you import `flask_sqlalchemy`:

    from flask_migrate import Migrate

Next, just after the bit where you have `db = SQLAlchemy(app)`, add this line to enable Flask-Migrate for your site:

    migrate = Migrate(app, db)

The first of these makes the migration stuff available, of course.   The second connects everything up so
that the migration system knows that it needs to be handling migrations for your app, specifically relating
to its connection to the SQLAlchemy database connection.

Save the file, then go back to your bash console.

Our next step is to create a directory where the migrations are going to live.  Remember, each migration is a
small Python file that defines how to make a change to the database.  We need to have somewhere to store them,
so run these two commands:

    export FLASK_APP=flask_app.py
    flask db init

It will print out a few lines saying that it's creating directories and files, with a warning at the end:

IMAGE HERE

We can ignore the warning for now.  Let's take a closer look at those two commands.   The second one first:

    flask db init

Because we have Flask installed, we have access to a bash command called `flask`.  It's a general utility script
for doing stuff with Flask code, and some extensions -- like Flask-Migrate -- hook into it to provide an easy way
to run their own commands.   The `db init` parameters tell it to execute some stuff provided by Flask-Migrate,
which does the file and directory setup for migrations that we wanted.

The first line:

    export FLASK_APP=flask_app.py

...sets an *environment variable* -- basically a variable inside Bash.   It's necessary because the `flask`
command doesn't necessarily know where the code for the Flask app you're asking it to work on might reside.
One thing that's worth knowing -- environment variables don't persist between Bash sessions, so if you start
a new Bash console -- or if the one you're working in is reset due to system maintenance on the PythonAnywhere
side, or something like that -- then you'll need to run the `export` command again to re-set it.   That said,
Flask won't do anything harmful if you don't have the environment variable set, it will just print out a
reasonably helpful error, like this:

    Error: Could not locate Flask application. You did not provide the FLASK_APP environment variable.

...so you'll know what you need to do.

OK, so we have our directory set up for migrations.

The next step is to create an "inital migration".   Because each migration -- each change that is made to the
database -- is represented by some Python code, we need to have an initial one that represents how to get from
an empty database to the state it was in before we started using Flask-Migrate.   This is actually a little
fiddly with Flask-Migrate when you have a database with some structure already, because it generates
each migration by looking at what's in the database, then looking at the code, and comparing the two.  Right
now the code and the database match up OK, so it won't know what to do.

However, there is a way -- what we want to do is tell it to compare the current code with an empty database, and
generate a migration that would create the appropriate structure in that database.

To do this, go to the PythonAnywhere "Databases" tab in a new browser tab, and create a new database called
`dummyempty`.  (In the unlikely event that you already have one with that name, just pick something else.)

Next, temporarily, we'll tell our Flask code to use the dummyempty database.  Go to the tab where your Python
code lives, and change the line

    databasename="yourdbusername$comments",

...to say

    databasename="yourdbusername$dummyempty",

Don't forget to replace `yourdbusername` with your actual MySQL username from the "Databases" tab. Save the
file, but *don't reload the website*.   This means that your site will keep running with the old code
while we sort out this database stuff.

Now we can generate the migration.  Go to your bash console, and run

    flask db migrate

You'll see something like this:

IMAGE HERE

The next step is to go back and change your code again so that it's not pointing at the wrong database --
change it back to saying

    databasename="yourdbusername$comments",

...then save the file.

So now we have a migration file (with a randomly-generated name) that contains Python code that knows how
to create our database structure -- not the contents! -- from scratch.  You may find it interesting to take a
look at the file -- it was the last thing printed out by the `flask db migrate` command.

There is one final step, though.   Flask-Migrate keeps track of which migrations have been applied to your
database -- it does this (in a kind of recursive, Inception-like manner) by keeping data in a database table in your
database to say what migration the database last had applied to it.

What we need to do is tell Flask-migrate "the stuff you see in the database right now is exactly what
correponds to the most recent migration you've generated -- don't worry about trying to add anything for it".
The command for this is pretty simple, if somewhat violent-sounding:

    flask db stamp head

Now, hopefully everything is in sync.  We can check that in two ways:

Firstly, we know that right now, we have one migration, which knows how to create the database structure that
is needed by the code that we currently have.   And we know that the database structure should match that exactly.
So if we ask Flask-Migrate to create a new migration, it should work out that nothing has changed in the code
to require a migration, so it should come back to us saying "I don't need to do anything".   Let's try:

IMAGE HERE

So that's good news.   Now let's check things out from a different angle.  Head over to the "Databases" tab,
and click the `yourusername$comments` link to start a MySQL database console.   When it's showing its
`mysql>` prompt, type

    show tables;

You'll see that as well as the `comments` table that holds our comments, we have a new one called
`alembic_version`.   Remember that Alembic is the underlying libary that Flask-Migrate uses to do its work.
Let's take a look at what it contains; run

   select * from alembic_version;

You'll see something like this:

IMAGE HERE

So there's single a `version_num` column, and one row with a hexadecimal number in the column.   What does that
mean?   Click back to your browser tab where you have the bash console, and look back a few commands to where you
ran the first `flask db migrate` command -- the one that generated the initial migration, not the one that did nothing.
You'll see that the file it created had a name with `_.py` at the end.   Before the `_.py`, there will be the same
hexadecimal number.   Likewise, when we ran `flack db stamp head`, it printed out a message to the effect that
it was running `stamp_revision` to that same number.

Basically, Flask-Migrate is tracking different migrations by giving them random hexadecimal
numbers as names, and then is storing them in files that contain that number, and storing the number in the database
to keep track of which ones it has applied.

OK, so now we have migrations working.  Once again, we've changed nothing visible -- but we're in the right place
to make the next step forward.   As always, let's save a checkpoint.   There are a couple of things we need to
do to make sure it's fully consistent.  Firstly, we need to update our `requirements.txt` so that we record
the fact that we now depend on Flask-Migrate.    Once again, run

    pip freeze > requirements.txt

...in your bash console, then edit the requirements file to remove the junk line from the start.

Next, commit the changes into git.  We need to add all of the new migration files as well as our changes to
the Flask app, so don't forget to `git add`:

IMAGE HERE including all git adds and the commit


## Finally adding the timestamp field

Now let's add our new field.  At last!  All we need to do for this is add a new line to the `Comment` class's
definition.  Right now we have this:

    class Comment(db.Model):

        __tablename__ = "comments"

        id = db.Column(db.Integer, primary_key=True)
        content = db.Column(db.String(4096))

...so just add a new line after the definition of the `content` field like this:

        posted = db.Column(db.DateTime, default=datetime.now)

What that's doing is probably pretty obvious, but sharp-eyed readers might spot that although the `default` is
being set to the method `utcnow` on the `datetime` class, there are no brackets after it to call that method.
That's deliberate!  By leaving out the brackets, we're saying that the default isn't the *result* of calling the
`now` function -- it's the function itself.   This is because all code that we put into a class like this is run
at the time the program is started.   So if instead we'd said

        posted = db.Column(db.DateTime, default=datetime.now())

...then every comment that was posted would have its `posted` timestamp set to the time the website was last
reloaded, which would be when the code `datetime.now()` was run.   By specifying the function instead, we tell
Flask-SQLAlchemy that this is something that needs to be run each time a new `Comment` object is created.
This is what we want.

Right, we have our new field.  Let's make the change to the database structure needed to support it.  Back in our
bash console, once again run the command to generate a migration:

    flask db migrate

Now, just like last time, that has generated a file that contains the code required to make a change to the
database -- specifically, to add a new column called `posted`.   But our database doesn't have that column
yet.   To actually make the change to the database, we run this:

    flask db upgrade

You'll see output like this:

IMAGE HERE

Now let's change our code to use the new column in the database.   Go to the page where you
have the Flask code, and change the line that renders the template back to one that just passes the
list of comments through:

    return render_template("main_page.html", comments=Comment.query.all())

Next, in the template, change the code that displays the timestamp so that it extracts it from
the comment object, by replacing this:

                        <small>Posted {{ timestamp.strftime("%A, %d %B %Y at %H:%M") }} by anonymous</small>

...with this:

                        <small>
                            Posted
                            {% if comment.posted %}
                                {{ comment.posted.strftime("%A, %d %B %Y at %H:%M") }}
                            {% else %}
                                at an unknown time
                            {% endif %}
                            by anonymous
                        </small>

What this does is display the timestamp for the comment if the comment has one, but displays
"at an unknown time" for comments -- like the ones in the database -- that don't have a timestamp.

Finally, reload your web app from the "Web" tab or from one of your editor tabs, and the reload the
site.   All of your existing comments will say that they were posted at an unknown time, but if
you add one, it will have the timestamp (in the "Universal" UTC timezone) set correctly:

IMAGE HERE

We've successfully added a new field to our comments, with a sensible value for the pre-existing
data.  It's `git commit` time!   Over in the bash console. run

    git status

...to see the changes, then

    git add .

...to add everything.   The `.` means "the current directory", so it's a quick way to stage all of
your changes for committing -- run

    git status

...again to confirm that the three files -- the Python code, the template and the migration -- have
all been staged, then

    git commit -m"Added a timestamp"

...to commit.


## Adding the commenter field


This is the last step; we want to store who it was that made each comment.   Now, we could cheat --
it would be really easy to add a string column to the database, and copy the username of the currently
logged in user over into it when a comment was posted.   But where's the fun in that?  Much too easy.
And also bad database design; right now we have this rather hacky setup where users are defined in the
code; like I said earlier, users should be kept in the database -- and if there are user objects
in the database, then it makes sense if each comment has a reference to the user that created it, surely :-)

So let's get our users into the DB.   The first step is to change our user class from being a normal
Python class (which happens to inherit from the Flask-Login `UserMixin` class to being one that
Flask-SQLAlchemy can deal with.   Firstly, change the class definition from

    class User(UserMixin):

to

    class User(UserMixin, db.Model):

This means that a `User` inherits not only the Flask-Login stuff that allows you to use it with that
package, but also the SQLAlchemy database stuff.   Now, because it's meant to be stored in the database,
we can't just specify the fields it contains in the `__init__` function; we need to add proper database
stuff for them (and a primary key for the database's internal use), and specify what the table is called:

    __tablename__ = "users"

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(128))
    password_hash = db.Column(db.String(128))

We can also get rid of the `__init__` function -- just delete this bit:

    def __init__(self, username, password):
        self.username = username
        self.password_hash = generate_password_hash(password)

...which means that you no longer need to import `generate_password_hash` at the top of the file.

We should also get rid of our naughty, insecure list of users with plaintext passwords -- delete this bit:

    all_users = {
        "admin": User("admin", "secret"),
        "bob": User("bob", "less-secret"),
        "caroline": User("caroline", "completely-secret"),
    }

The `load_user` function needs to be changed to check the database:

    @login_manager.user_loader
    def load_user(user_id):
        return User.query.filter_by(username=user_id).first()

What we're doing here is using SQLAlchemy to query the database, and get the first item that has the given
username.   It will return `None` if there is no user with the given username.

Finally, we need to change the code in `login` to check the list of users from the database.  Where we currently
have

    username = request.form["username"]
    if username not in all_users:
        return render_template("login_page.html", error=True)

    user = all_users[username]


...we should do this:

    user = load_user(request.form["username"])
    if user is None:
        return render_template("login_page.html", error=True)

...using our `load_user` function from earlier to get the User object, and relying on it returning
`None` for nonexistent users.

So now we've done the code changes; we need to add the table to contain users to the database.  This
is a simple database migration.   Head over to the Bash console, then run

    flask db migrate

...and then

    flask db upgrade

Now let's add our users back in.  Instead of putting them in code (where their passwords are
available in plaintext for all to see), we'll add them from a command line.   To start a Python
shell with all of the Flask stuff loaded, run this:

    flask shell

In the Python interpreter that appears, firstly import our User class and the database connection:

    from flask_app import db, User

Next, import the password-hashing function that we were previously using in the code:

    from werkzeug.security import generate_password_hash

Finally, add the users you want, using commands like the ones below but with different passwords:

    admin = User(username="admin", password_hash=generate_password_hash("secret"))
    db.session.add(admin)
    bob = User(username="bob", password_hash=generate_password_hash("less-secret"))
    db.session.add(bob)
    caroline = User(username="caroline", password_hash=generate_password_hash("completely-secret"))
    db.session.add(caroline)
    db.session.commit()

Hit control-D to exit the shell.

Now, go over to the "Web" tab to reload the website.  It should work exactly as before, but when you
log in, you'll need to use the new passwords.   Our users are now stored in the database!

Let's checkpoint -- `git status` to see what there is to add -- it should be a new migration file and
some changes to `flask_app.py` -- `git add` to add them, and `git commit` to finish off.

The final step is to set up an association between comments and users.  This needs two fields to
be added to the `Comment` class:

    commenter_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=True)
    commenter = db.relationship('User', foreign_keys=commenter_id)

The first of these says that `Comment` objects have a field called `commenter_id`, which is the
database ID of the commenter.  The second sets up a special field on `Comment` so that if we just
ask for `comment.commenter`, it will look up the user by the ID in `commenter_id`.

Save the file, then head over to the bash console, generate the migration, and upgrade the database:

IMAGE HERE

Now we want to save the commenter when the comment is first created; where we currently have

    comment = Comment(content=request.form["contents"])

...we should put this code:

    comment = Comment(content=request.form["contents"], commenter=current_user)

Now, over in the template, let's use the commenter's name when displaying the comments -- or, if
there is none, we'll put in an alternative name that will be familiar to anyone who read Slashdot
back in the 90s (he said, showing his age).   Replace the

                            by anonymous

bit in the comment rows with this:

                            by
                            {% if comment.commenter %}
                                {{ comment.commenter.username }}
                            {% else %}
                                Anonymous Coward
                            {% endif %}

Reload the site once again from the "Web" tab, and visit it.   You'll see that all of the existing comments
are now showing up as having been posted by "Anonymous Coward".   Add a new one -- and you'll see that
the username is stored :-)

All fairly easy, right?   Setting up the virtualenv and Flask-Migrate did take a bit of time and effort
(you can probably see why we didn't include them in the first tutorial -- it's too much to dump on someone
when they're a complete beginner), but it does pay off quickly.

So, there's just one more thing to do:

IMAGE SHOWING TRIUMPHAL FINAL COMMIT

## That's all, folks!

That's it for this tutorial.   We hope you've found it useful, and if there's anything that confused you,
or you encountered any errors, please do leave a comment below and we'll help you out.

One final thing -- many thank to Miguel Grinberg for creating Flask-Migrate, and in particular for
[helping out](https://github.com/miguelgrinberg/Flask-Migrate/issues/31#issuecomment-320246309)
so quickly when we were trying to discover how to get it to work with an existing database.



