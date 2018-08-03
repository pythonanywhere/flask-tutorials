Welcome to the third part in our tutorial for getting started with Flask development on PythonAnywhere! This is a continuation of our previous tutorial ([part 1](https://blog.pythonanywhere.com/121/), [part 2](https://blog.pythonanywhere.com/158/)) and builds on the codebase we worked on there, so if you haven't been through those, you should do that first.

So at the end of the last part of the tutorial, we had a website that showed a list of comments posted by people who could log in to the site.   Only authorized users could post comments, and list of authorized users was stored in the database.   And, of course, all of our source code was under source-code control in a git repository that is stored in our private file storage on PythonAnywhere.

In this part of the tutorial, we're going to pretend that our site now has lots of people using it regularly.   There are two knock-on effects from that:

* It would be really boring work to have to keep using the `flask shell` command that we used last time to add new users (and anyway, how would they contact us to ask to be added to the site?)
* We really don't want to break the site, even temporarily.   So far, when we've been making changes to it, we've just gone right in and hacked the code directly.   If we have a lot of users for a site, that won't be acceptable.

So this time around we're going to fix the second one of those first, by creating a completely new copy of the site
in a second PythonAnywhere account.  This will be our "development" environment, where we can hack away to our hearts' content while adding new features, knowing that our thousands of happy users won't be inconvenienced.

And once we've got that ready, we can fix the first problem, by adding a registration form where people can sign up to join the site.   They'll enter their name, email address and password, we'll do some simple email validation, and then once that's done they can post away.

Here's what it will look like:

IMAGE

Sound good?   Let's get going :-)


## Setting up a separate development environment

Because you can only have one website on a free PythonAnywhere account, but we want a separate one for our
development environment, we'll need a second account.   That's not a problem -- you can sign up for multiple
free accounts with the same email address.

(If you already have a paid PythonAnywhere account -- thanks! -- then you could in theory do all of this in
one account with multiple websites.   But then you'd miss out on all of the cool git stuff that's coming up,
so please do try doing it this way -- you won't regret it :-)   Also, you could in theory set up a development
environment on your own local computer; the techniques would be pretty much the same.   I'll stick to explaining
it in terms of separate PythonAnywhere accounts, though, as it makes the explanation simpler -- and if you want
to move over to doing stuff from your own machine later, you'll know enough of the details to make it pretty
easy to set up.)

Now, the development process that we want to set up is to have

* The "live" site running in our existing PythonAnywhere account, with all of our thousands of users happily using it.
* The "dev" site in a new account, with no-one using it apart from us.

We'll make changes in dev, test them (against a separate database so that our test comments don't pollute the
live site), and then move them from dev to live -- a process normally called "deployment".

The obvious question is, how do we move the code from dev to live?   You could copy files around or use FTP or
something like that, but it would be tedious and error-prone.   However, we're already using a tool that can
make it safe and easy -- git.

All of our code is stored in a git repository that is stored in our private file storage on PythonAnywhere.
Now, one nice thing about git is that you can push code from one repository to another -- it's this
functionality that has made it so popular; it was originally developed to make life easier for the Linux
kernel developers; with a large number of people working on different copies of the code, they needed
a tool so that they could pass proposed code changes around easily.   Using git for complex workflows
like that can be tricky, but for a simple case like ours, with two environments that we want to copy
changed between, it's actually really easy.

There is one wrinkle, though.   We could set things up so that we could simply push and pull changes between
our dev and live environments directly, but it would be a bit tricky to do that in a free PythonAnywhere account.
So instead, we're going to use GitHub.

If you're not familiar with GitHub, don't worry -- although it does a lot of great stuff, we're only going to
use a simple subset of its features.   Basically, you can see it as a place where you can store repositories
of code.   If you have a repository on GitHub, you can push code to it from one machine, and pull it down from
another -- so it can act as a central place that we use to move code around.   A diagram should help:

IMAGE

So we're going to create a repository on GitHub, then set things up so that it can receive code from
the one we already have on PythonAnywhere, then ee'll push our existing code up there.

Once that's done, we'll pull the code down from GitHub into the PythonAnywhere account we're going to create
for the dev environment.   We'll set up the dev environment, and make sure that we can have a running copy
of our site there.  Once that's done, we'll be able to make changes in dev, test them, push them up to GitHub,
and then pull them down into live to deploy the changes.

So let's do all of that then :-)

The first step will be to make sure that our code is something that we can safely put on GitHub.   Right now
it's not :-(    This is because with a free GitHub account, your code repositories are visible to anyone in the
world -- and we have our database password and the Flask "secret key" right there in the code.   Whoops!

Sidebar: it's actually bad practice to put passwords and secrets in a repository, even if you plan to keep
it private -- we shouldn't have
done it in the first part of the tutorial.   But on the other hand,
doing this stuff at the start would have made the first tutorial hard, and
so we've delayed it until now.   Also doing it now and emphasizing that this is fixing a mistake maybe
will make it easier to remember.  Or, at least, that's my story and I'm sticking to it :-)

The way we'll sanitise our repository is to move all of the private stuff into a new private file -- that
is, one that we won't put in the repository.


Move SQLALCHEMY_DATABASE_URI, SECRET_KEY into private_settings.py

Add from private_settings import SQLALCHEMY_DATABASE_URI, SECRET_KEY to flask app

Reload, check it works.

Now, because the history of our repository contains the old secret and mysql password we
need to cahnge them




Change the SECRET_KEY

Change the DB password on the databases page, and change the URI appropriately

Reload, check.

Now add private_settings.py to .gitignore

Commit

So now we can safely share the code with other people without them knowing our password or secret key

Explainer on github push/pull

Let's sign up for a github account.   If you already have one, you can use it, of course

Once we're in, click the "New repository" button.

This will create a publicly-visible empty git repository on GitHub.  Make the repository name something like "flask-tutorial".

It will now display a page like this.    These are the options for connecting a repository


Note the "...or push an existing repository from the command line".   We have our repository on PythonAnywhere,
and we want to connect it up with this one on GitHub.   Later well create a new one on our development account
and then pull the repository down to there.

So copy/paste the two lines into your bash console.   The second one will prompt you for your github username and password

Now go back to github -- you can see your code in their interface.

The next step is to create a new PythonAnywhere account for your development site.

Log out of PythonAnywhere, and sign up for a new free account

In the new account, create a new Python 3.6 website

Check that it displays the "Hello from Flask" message

Start a bash console

We're going to get our code down from GitHub.  Firstly -- the code for our new website is
just stored in a directory called mysite, which we want to replace.   Here's the command in bash

    rm -r mysite

Now we clone it.   In the github window, there's a "clone or download" button.  Click it, and a
thing will pop up with "clone with HTTPS" and a URL like https://github.com/flasksimpletutorial/flask-tutorial.git
-- copy that URL.

Back in the bash console, run

    git clone https://github.com/flasksimpletutorial/flask-tutorial.git mysite

-- that is, clone the code from that URL into a directory called mysite

go into the directory (cd mysite) and run git log -- you'll see that all of your history is there,
just like in the original account

Now we need a virtualenv.   In bash, run this (the same as we did from the other account)

    mkvirtualenv flask-tutorial --python=python3.6

Now we'll install our requirements.

    cd mysite
    pip install -r requirements.txt

When it's done, on the "Web" page, type "flask-tutorial" into the virtualenv field (under code)
and reload the website.   It'll be broken -- no database.

Now we need a database for this version of our site.   Best to have dev and live on different databases so that we don't
break stuff while testing.

On the Databases page, initialize MySQL.   Use a different password to the one in the other account.

Once it's all initialized, create a database called "comments", just like the one in the live site.

Our site will need a private_settings.py.   On the "files" page, create an empty file with that name inside
mysite.

Put appropriate contents in it -- remember, it should look something like:

    SQLALCHEMY_DATABASE_URI = "mysql+mysqlconnector://{username}:{password}@{hostname}/{databasename}".format(
        username="SOMETHING",
        password="SOMETHINGELSE",
        hostname="SOMETHING.mysql.pythonanywhere-services.com",
        databasename="SOMETHING$comments",
    )
    SECRET_KEY = "SOMETHING RANDOM"

Generate the secret key with the normal random key-bashing; the username should come from the databases page,
as should the hostname; the password should be the one you entered there.   Also remember that you need the
username from the databases page before the $ in the database name.

First we need to tell the Flask tools where our code is

    export FLASK_APP=flask_app.py

....and now we can just upgrade the database, which will run all of the migrations we painstaking created in the last tutorial so that all of the appropriate tables are there.

    flask db upgrade

So now we have a site with a database and a virtualenv.

Visit the site -- it'll all be up and running!   But we can't log in, as there are no users.

>>> from flask_app import db, User
>>> from werkzeug.security import generate_password_hash
>>> admin = User(username="testuser", password_hash=generate_password_hash("youllneverguess"))
>>> db.session.add(admin)
>>> db.session.commit()

Now back to the tab showing the website; we can log in with testuser and the given password.

Try adding a comment -- it's there!

Check out our old site -- of course, the comment isn't there.

So now we have a development version of our site that we can hack on without having to worry
about inconveniencing the thousands of keen users of our main "live" site.

Before we dive in and start making big changes, let's try making a nice simple change so that
we can go through the deployment process once.   Because we now have thousands of users for our
site, calling it "My scratchpad" is a bit egotistical.   Let's call it "Our scratchpad" :-)

Still in our dev PythonAnywhere account, go to the "Files" page and open up the "mysite" folder,
then go into the "templates" directory.   Open up, in turn, the login_page.html and the main_page.html
files, and change the "title" tags to reflect our new, more-open site name.

CODE SAMPLES

Visit the site (remember, you don't need to reload if it's just a template change) and you'll see it's updated.
But, of course, these changes only exist on the development version of the site.

Let's deploy!

First, commit the changes.  Firstly, because this is a new PythonAnywhere account, we need
to tell git who we are.   Back to the bash console and run the two commands you did in the other
account, way back in part 1 of the tutoria (replacing the stuff in the quotes appropriately):

    git config --global user.name "Your Name"
    git config --global user.email "you@example.com"

We'll also need an extra bit of configuration:

    git config --global push.default simple

(This is simply to disable a bothersome message git will print out.)

Now you can commit:

    git add .
    git commit -m"Changed title to be more inclusive"

So the changes we've made exist in the local git repository, stored inside this PythonAnywhere account --
but nowhere else.    Run git log to see the history

Then, head over to the tab where you kept GitHub open -- click down through the templates directory and look at the main_page.html -- you'll see that the changes haven't propagated there yet, which makes sense because we haven't done anything to make that happen.

So the next step is to push the changes to GitHub.   This is really simple:

    (flask-tutorial) 18:01 ~/mysite (master)$ git push
    Username for 'https://github.com': flasksimpletutorial
    Password for 'https://flasksimpletutorial@github.com':
    Counting objects: 9, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (5/5), 451 bytes | 0 bytes/s, done.
    Total 5 (delta 3), reused 0 (delta 0)
    remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
    To https://github.com/flasksimpletutorial/flask-tutorial.git
       a0ef303..819bc87  master -> master

Now check out the code on GitHub -- you'll see the changes there.

The next step is the opposite side of the push we just did -- we want to pull the changes down
into the live site.

Now, we could just log out of PythonAnywhere and log back in to the original account, but that could
get boring quite quickly if we did it every time we did a deploy -- and hopefully we'll be doing
a number of deployments over the course of this tutorial.   One way of avoiding that is to make use of
your browser's incognito mode; you can have two PythonAnywhere sesssions active, one in your browser's
normal mode and one in incognito.

So, open a new incognito window, and in it, go to PythonAnywhere.   From there, log in as your original
user, the one that owns the live site.   Got to your bash console, make sure you're cd'ed into the mysite
directory, and do a "git pull":

    16:51 ~/mysite (master)$ git pull
    remote: Counting objects: 5, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 5 (delta 3), reused 5 (delta 3), pack-reused 0
    Unpacking objects: 100% (5/5), done.
    From https://github.com/flasksimpletutorial/flask-tutorial
       a0ef303..819bc87  master     -> origin/master
    Updating a0ef303..819bc87
    Fast-forward
     templates/login_page.html | 4 ++--
     templates/main_page.html  | 4 ++--
     2 files changed, 4 insertions(+), 4 deletions(-)
    18:08 ~/mysite (master)$

The templates were updated; you can confirm that by going to the website, and you'll see that the
titles have changed.

So now we've created an independent development environment for our website, and successfully set everything
up so that we can easily push changes from dev through to live.

Now let's get started with some more serious changes.




thoraway1234