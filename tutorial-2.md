Database-backed Flask development on PythonAnywhere: database migrations and virtualenvs

Welcome to the second part in our tutorial for getting started with Flask development on PythonAnywhere!  In this part we'll explain two new concepts, both of which are important for Python web development: virtualenvs, which allow you to control which external Python packages your code uses, and easily use new ones, and database migrations, which make it easy to change what you're storing in a database without losing the stuff you already have stored there.

If you haven't already read [the first part](https://blog.pythonanywhere.com/121/), we recommend you run through that first -- we'll be building on the website that was created there.   As a reminder, we had a simple website that allowed you to post comments on a page, all of those comments being stored in a database.

<img width="500" src="/static/images/flask-tutorial-final-result.png">

It was made up of two files; a Python one which controlled what it did, and a template one that controlled how it was displayed.  All of our code was under source-code control, and was running as a website on PythonAnywhere.

The main visible change that we're going to get out of this tutorial is that each comment will have an associated timestamp.   Not an enormous difference, it's true :-)   But getting there requires learning enough new stuff that it's probably quite enough for one session of learning.

## virtualenvs

Virtualenvs were mentioned a couple of times in the last part of the tutorial, but we kind of glossed over what they were.  Now's the time for a bit more detail.

## Why?

The Python programming language has a bunch of packages build in, the ["Standard Library"](https://docs.python.org/3.6/library/index.html), which do a lot of useful things.  However, most code that you write will probably use packages that aren't officially part of Python -- they're separate open-source packages, which are maintained by separate groups of people.   The code you wrote in the first part of the tutorial already uses some of them -- [Flask](http://flask.pocoo.org/) is one such package, and so is [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.1/) -- and the underlying SQLAlchemy package that [Flask-SQLAlchemy](http://www.sqlalchemy.org/) depends on.  There's an official website that lists almost all of the popular Python packages, the [Python Package Index](https://pypi.python.org/pypi), normally abbreviated to PyPI.

If you were to install Python on your own machine, you'd just get the standard library; if you wanted Flask and the others, you'd need to install them separately.   PythonAnywhere, in order to make it easy to get started, has a [*lot*](https://www.pythonanywhere.com/batteries_included/) of them pre-installed.  But it doesn't have everything (there are 111,690 packages on PyPI as of this writing, so you can see how that might be a bit impractical), and for this bit of the tutorial we're going to need one that isn't there.

The normal way to install packages from PyPI is using a tool called `pip`.  It can install things three different ways:

1. For the entire system, so that every Python program you run has access to it.  Unfortunately you can't do this on PythonAnywhere -- everyone has the same system image.
2. Just for your own user account, so just your programs (and no-one else's) have access to it.  Other people on the system would have to install it too in order to use it.  This is possible on PythonAnywhere, but it's not the best way, which is:
3. Into a virtualenv.  A virtualenv, or "virtual environment", is a separate directory in your home directory that contains non-system packages that you've chosen to install into the virtualenv.   When you run some Python code (or in your website setup) you can say "use the virtualenv called X", and then that code will have access to the standard library and the virtualenv's packages -- nothing else.

So why is that useful?  It's basically because code changes over time.  You can be pretty certain that the Python core developers won't change things drastically in a new version in a manner that will cause your code to stop working -- or, at least, they won't do it without plenty of warning.   But the same doesn't necessarily hold for other packages.

*(I'm going to use Flask as an example of a rapidly-changing package below, which isn't entirely fair -- the Flask team are super-careful about not breaking people's stuff without warning.  But the risks still hold -- maybe you might miss their announcement that they were going to change things.)*

Let's say you create a website based on Flask 0.12.  You have it up and running, and people are using it.   One day, you decide to write a new, separate website.   You've heard that the latest version of Flask, 0.18, has lots of great new stuff in it, so you install it on PythonAnywhere so that you can use the new things.

Unfortunately, what you didn't realise was that 0.18 wasn't backward-compatible with 0.12 -- that is, the code that you wrote with 0.12 doesn't work with 0.18.   Your popular site is broken, and people start sending you grumpy emails.  So now you need to either roll back to the older version of Flask and write your new site using 0.12, or you need to change all of your old code to work with the new version.

If you've not done much development before, this may sound like a kind of abstract, "let's worry about it later" kind of problem.   And it is, and would be... right up until the point you found yourself frantically dealing with version compatibility problems and cursing the day you thought it might be a good idea to learn to code.

So how would a virtualenv have helped in this case?  Well, when you wrote your first app, you would have created a virtualenv for it.  Into that virtualenv, you'd install Flask 0.12, and you'd have configured the website to use that virtualenv.

When you started work on your new website later, and decided you wanted access to all of that Flask 0.18 goodness, you'd create a completely new virtualenv, install 0.18 into that, and set up the new website to use it.  The two would be isolated from each other.   Disaster averted!

### OK, let's try it.

That's probably more than enough background.   What we're going to do is create a virtualenv, install the stuff that we need in order to run our Flask website into it, then switch the Flask app over to using it.   Later on we'll install things into it.

## Creating a virtualenv

The first step is, of course, to log in to PythonAnywhere.  Start a bash console, and run the following command:

    mkvirtualenv flask-tutorial --python=python3.6

It will take a little time to run.   The command `mkvirtualenv` comes from a helpful mode called `virtualenvwrapper`; virtualenvs can be a little fiddly to work with if you use the normal tools, and virtualenvwrapper helps with that.  The parameters that we've given it are first, a name for our virtualenv, `flask-tutorial`, and secondly a specification of which version of Python we want to use -- any given computer can have multiple versions installed (PythonAnywhere has a bunch, unsurprisingly) and a virtualenv normally only supports one, so you need to be clear which one you want to use.

Once the command has completed, you'll notice that the prompt has changed a bit.   Before it would have been something like this:

    17:39 ~ $

But now it will look like this:

    (flask-tutorial) 17:42 ~ $

When you're in a virtualenv, the prompt shows you its name at the start.  This is very useful if you have lots of them, of course.

Now, our next step is to install some packages using `pip`.   There are three packages that we need to install; Flask itself, the Flask-SQLAlchemy plugin, and mysqlconnector, a library that helps SQLAlchemy talk to MySQL.  They also have various dependencies -- Flask depends on a templating library called Jinja, for example -- but you don't need to worry about those -- `pip` will automatically install them for you.   So all you need to do is run this command:

    pip install flask flask-sqlalchemy mysql-connector-python

That will take a little while to run.   It will pull down all of the packages from PyPI, along with their dependencies, and install them into your virtualenv.   When it's finished and you get the bash prompt back, you can see what it's installed by running the `pip freeze` command.  You'll see something like this:

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

You can see that it's showing you not only the packages you installed, but allso the dependencies, and it's showing the version number for each one.

So, we have a virtualenv.  But our website is still using the packages that were installed system-wide, so it's not being used.  The next step is to fix that.  Open a new tab by right-clicking the PythonAnywhere logo, and go to the "Web" tab.   Make sure that your website is selected, then scroll down a bit.   You'll see a section with the header "Virtualenv":


Click on the link to edit the field, and enter the name of the virtualenv you created earlier, `flask-tutorial`.  The system will think for a second, and then fill in the full path to the virtualenv.  A virtualenv is just a directory in your private file storage -- you normally don't need to worry about where it is, though, you can just use the name.

Now that we've made a change on the "Web" tab, the next step is to reload the website to make that change take effect.  Scroll back up to the top of the page, and click the green "Reload" button.  Wait for the spinner to disappear, and then right-click your site's name to visit it.  If all went well, it should display just like it did before, showing all of the comments that were already there.  Hooray!

Well, maybe not quite "hooray", as all we've done is avoid breaking things -- however, sometimes that's just how coding works ;-)

Now, it's good practice to keep a list of the external packages that your code depends on inside your Git repository.  The tradition is to use a file called `requirements.txt`.   In order to avoid having to type everything in, let's use a bash command.  Go back to your bash console.   If you started it from the "Consoles" tab, then its working directory will be your home directory, which is normally abbreviated to `~` -- check the bash prompt, and it will look like this:

    (flask-tutorial) 17:55 ~ $

As the source files for the website are in the subdirectory `mysite`, the first step is to change our working directory using the command `cd mysite`, which will leave us with a bash prompt like this:

    (flask-tutorial) 18:05 ~/mysite (master)$

Note that `~` has been replaced by `~/mysite`.  (There's also that `(master)` that's appeared, which is because you're in a git repository -- more about that in a later tutorial, perhaps.   You can ignore it for now.)

So, we want to create a file called `requirements.txt` that lists the packages we use.  And we already know there's a command `pip freeze` that will produce that list.  So we can run the command and tell it to write its output to a file:


    pip freeze > requirements.txt

If all goes well, that command will just send you back to the prompt.   Now, open another tab by right-clicking the PythonAnywhere logo, and go to the "Files" tab, then down into the `mysite` directory.  You'll see the file `requirements.txt` -- click on it to load it into the editor.

What we want to do here is remove the first line, the one that says `-f /usr/share/pip-wheels`; it's not actually one of our requirements, it's just some extra junk that pip has printed.  So remove that line, then save the file.

Now let's get the requirements file into our repository.   If you run `git status` you'll see that `requirements.txt` shows up in red as an "untracked file" -- remember, git needs to be explicitly told that it should track new files.   So, use `git add` to add it to the list of tracked files, then `git commit` to take a checkpoint and add it to the repository.

## Designing timestamps

Again, all we've done so far is change how the website works without breaking things -- but small, non-breaking changes are often a good plan when coding, as it helps you move forward in the direction you want to go without leaving everything in a messy state if you get stuck.

But now it's time to actually make some changes to the site.  Our goal is to add a timestamp to every comment; like we did before, we'll start off by changing the template so that we can see what it looks like, then we can make the back-end Python changes required to make it a Real Thing.

Open up a browser tab, and load up the Python code.  We'll start off by passing the current time to the template and rendering that for every comment, just so we can get a feel for what it will look like.   To do that, add this line to the top of the file to import the `datetime` class from the standard library:

    from datetime import datetime

...then change the code that renders the template from this:

    return render_template("main_page.html", comments=Comment.query.all())

...to this:

    return render_template("main_page.html", comments=Comment.query.all(), timestamp=datetime.now())

Now, reload the web app and take a look at the site -- once again, it won't have changed.  This is a good thing :-)

Next, in a new browser tab, open the template file.  Right now, we have a `for` loop and inside it, for each comment, we have this HTML:

                <div class="row">
                    {{ comment.content }}
                </div>

Replace it with this code:

                <div class="row">
                    <div>
                        {{ comment.content }}
                    </div>
                    <div>
                        <small>Posted {{ timestamp.strftime("%A, %d %B %Y at %H:%M") }}</small>
                    </div>
                </div>

Check out the site again -- now you'll see that each comment has a timestamp next to it, saying something like "Posted Friday, 07 July 2017 at 18:47".   Because we're not storing the timestamps yet, all of them will have the same timestamp -- the time at which the page was rendered.  But it's a start!

If you prefer US-style date formatting, where the month comes before the day, the change is really simple -- just swap around the `%d` and the `%B` in the template where it says

    {{ timestamp.strftime("%A, %d %b %Y at %H:%M") }}

What that line means is, call the `strftime` method on the `timestamp` object that we're passing in to the template.  You can read more about what all of the different options for the `%` bits in there are on [this Python documentation page](https://docs.python.org/3/library/time.html#time.strftime).

Once you have everything looking the way you want, let's take a checkpoint.  Go to your bash console, then use `git diff` to check that you've only got changes that you expect to see, and then commit them using:

    git commit -am"Design for timestamps"

## Implementing timestamps

Now it's time to write the code to actually save timestamps.  This is a little tricky.  Right now, we have a number of comments on our site.  Those comments don't have timestamps, and even worse, they're stored in a database table that doesn't have a comment where we could store timestamps.  Now, of course, for a website we're just building to play with like this one, we could just delete everything and build the database up from scratch, this time with a timestamp field.   But for a real site that would be a bit destructive.

What we need to do is change the database in a controlled way.   We want to be able to add a new timestamp field to our `Comment` class, which is as easy as you'd expect, but we also want to be able to change the database to add the new column, putting a sane value in there for the existing comments.

It is actually possible to do that by starting a MySQL console and changing things by hand, but it's a lot of hassle -- and doesn't work very well when you're making regular changes as you extend your site.   So we're going to use a tool called migrations.   A migration is, conceptually, a change to a database's structure packaged up as a small Python file that knows how to make the change.

There's a Flask extension module that handles all of this stuff for you very nicely, so let's get it installed into our virtualenv (so now we actually get some benefit from the first step ;-)   Go to your bash console, and run this:

    pip install Flask-Migrate

That will take a minute or two to install, as Flask-Migrate depends on a much larger module called `alembic`, which actually does all the migration stuff.

Once it's all installed, we need to configure our code to use it.  We do this *before* we make any changes to the database, because we're going to have to run some commands first in order to let the system know what the current state of the database is before we start modifying things.

In your Python code editor, add a new line just underneath the bit where you import `flask_sqlalchemy`:

    from flask_migrate import Migrate

Next, just after the bit where you have `db = SQLAlchemy(app)`, add this line to enable Flask-Migrate for your site:

    migrate = Migrate(app, db)

Save the file, then go back to your bash console.

Our next step is to create a directory where the migrations are going to live.  Remember, each migration is a small Python file that defines how to make a change to the database.  We need to have somewhere to store them, so run these two commands:

    export FLASK_APP=flask_app.py
    flask db init

It will print out a few lines saying that it's creating directories and files, with a warning at the end:


Before we move on to that, let's take a closer look at those two commands.   The second one first:

    flask db init

Because we have Flask installed, we have access to a bash command called `flask`.  It's a general utility script for doing stuff with Flask code, and some extensions -- like Flask-Migrate -- hook into it to provide an easy way to run their own commands.   The `db init` parameters tell it to execute some stuff provided by Flask-Migrate, which does the file and directory setup for migrations that we wanted.

The first line:

    export FLASK_APP=flask_app.py

...sets and *environment variable* -- basically a variable inside Bash.   It's necessary because the `flask` command doesn't necessarily know where the code for the Flask app you're asking it to work on might reside.   One thing that's worth knowing -- environment variables don't persist between Bash sessions, so if you start a new Bash console -- or if the one you're working in is reset due to system maintenance on the PythonAnywhere side, or something like that -- then you'll need to run the `export` command again to re-set it.   That said, Flask won't do anything harmful if you don't have the environment variable set, it will just print out a reasonably helpful error, so you'll know what you need to do:


    Error: Could not locate Flask application. You did not provide the FLASK_APP environment variable.

OK, so we have our directory set up for migrations.   Now, Flask-Migrate told us:

    Please edit configuration/connection/logging settings in '/home/yourusername/mysite/migrations/alembic.ini' before proceeding.

However, to be honest you don't really need to.  The default settings are all pretty good, and will certainly suffice for now.  So let's move on...

The next step is to create an "inital migration".   Because each migration -- each change that is made to the database -- is represented by some Python code, it's useful, and good practice -- though not entirely necessary -- to have an initial one that represents the state of the database before we started using Flask-Migrate.   This is actually a little fiddly with Flask-Migrate when you already have a database with some structure already, because it generates each migration by looking at what's in the database, then looking at the code, and comparing the two.

However, there is a way -- what we want to do is tell it to compare the current code with an empty database, and generate a migration that would create the appropriate structure in that database.  If you've been following the tutorial exactly so far, you'll have an empty one to do that with -- in a new brower tab, go to the "Databases" page, and check the "Your databases" list.  You'll see two, a default empty one called *yourusername*`$default`, and the one we're using for the website, called *yourusername*`$comments`.

(If you've deleted that database, or if you've put things into it, you can just create a new database and then use it below.)

So, temporarily, we'll tell our Flask code to use the "default" one.  Go to the tab where your Python code lives, and change the line

    databasename="flasksimpletutor$comments",

...to say

    databasename="flasksimpletutor$default",

Save the file, but *don't reload the website*.   This means that your site will keep running with the old code while we sort out this database stuff.

Now we can generate the migration.  Go to your bash console, and run

    flask db migrate

You'll see something like this:


The next step is to go back and change your code again so that it's not pointing at the wrong database -- change it back to saying

    databasename="flasksimpletutor$comments",

...then save the file.

So now we have a migration file (with a randomly-generated name) that contains Python code that knows how to create our database structure -- not the contents! -- from scratch.  You may find it interesting to take a look -- it should be pretty clear.

Once again, we've changed nothing visible -- but we're in the right place to make the next step forward.   As always, let's save a checkpoint.   We need to add all of the new migration files as well as our changes to the Flask app, so don't forget to `git add`:


## Finally adding the field

Now let's add our new field.  At last!   All we need to do for this is add a new line to the `Comment` class's definition.  Right now we have this:

    class Comment(db.Model):

        __tablename__ = "comments"

        id = db.Column(db.Integer, primary_key=True)
        content = db.Column(db.String(4096))

...so just add a new line after the definition of the `content` field like this:

        posted = db.Column(db.DateTime, default=datetime.now)

What that's doing is probably pretty obvious, but sharp-eyed readers might spot that although the `default` is being set to the method `utcnow` on the `datetime` class, there are no brackets after it to call that method.   That's deliberate!  By leaving out the brackets, we're saying that the default isn't the *result* of calling the `now` function -- it's the function itself.   This is because all code that we put into a class like this is run at the time the program is started.   So if instead we'd said

        posted = db.Column(db.DateTime, default=datetime.now())

...then every comment that was posted would have its `posted` timestamp set to the time the website was last reloaded, which would be when the code `datetime.now()` was run.   By specifying the function instead, we tell Flask-SQLAlchemy that this is something that needs to be run each time a new `Comment` object is created.   This is what we want.

Right, we have our new field.  Let's make the change to the database structure needed to support it.  Back in our bash console, run this:






