---
title: Openshift V2 and Flask with Virtualenv on OS X
banner: /img/banners/banner-1.jpg
date: 2014-11-22
layout: post
tags:
 - python
 - openshift
 - mac
---

Create a flask app for Openshift with a matching local python virtualenv to perform local testing.

In this case we'll use Python 2.7 on Mac OS X 10.9.

Overview of the steps
---
1. Install [Homebrew](http://hackercodex.com/guide/mac-osx-mavericks-10.9-configuration/)
2. Install [Python Development Environment on Mac OS X](http://hackercodex.com/guide/python-development-environment-on-mac-osx/)
3. Install [rhc client tools](https://www.openshift.com/developers/rhc-client-tools-install).
4. Install [and Configure a Python Flask for OpenShift](https://www.openshift.com/blogs/how-to-install-and-configure-a-python-flask-dev-environment-deploy-to-openshift)

<!-- break -->

Installing Homebrew
===

Ready the system for Homebrew
---
- First unhide ~/Library folder. 
	1. Open Finder
	2. Press *shift-command-H*
	3. Press *command-J*
	4. Check *Show Library Folder*

- Now Setup shell environment
Some of these settings are only relevant to later steps, but go ahead and put them all in now.
 - vim ~/.bash_profile

```bash 
# Set architecture flags
export ARCHFLAGS="-arch x86_64"
# Ensure user-installed binaries take precedence
export PATH=/usr/local/bin:$PATH

# Load brew bash completeion if it exists
if [ -f $(brew --prefix)/etc/bash_completion ]; then
    . $(brew --prefix)/etc/bash_completion
fi

# pip should only run if there is a virtualenv currently activated
export PIP_REQUIRE_VIRTUALENV=true
# cache pip-installed packages to avoid re-downloading
export PIP_DOWNLOAD_CACHE=$HOME/.pip/cache

# Load .bashrc if it exists
test -f ~/.bashrc && source ~/.bashrc
```

- Source those changes. Ignore the *-bash: brew: command not found* error.

```bash 
. ~/.bash_profile
```

- Install command line developer tools or Xcode. You'll need to be on the actual console, so don't do this step over SSH.

```bash 
xcode-select --install
```

This will prompt you with a GUI dialog asking you to install the command line developer tools. Click the *Install* button.


Install Homebrew
---
- Install Homebrew

```bash 
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- Inspect and update brew install. 

```bash 
# This step will possibly point out permissions problems to be fixed.
brew doctor
brew update
brew help
```

- Install some brew packages
```bash 
brew install bash-completion ssh-copy-id wget
```

Install Python
===
Install the latest 2.7.x Python with Homebrew.

```bash 
brew install python --with-brewed-openssl
```

Setup Virtenv
===

Use pip to install virtualenv.

```bash 
# override the requirement we set in .bash_profile above. just this once.
PIP_REQUIRE_VIRTUALENV=false pip install virtualenv
mkdir -p ~/src ~/Projects ~/Virtualenvs
```
 
Setup Openshift
===

Interaction with Openshift is via [the website](http://www.openshift.com) and via the [rhc client tool](https://www.openshift.com/developers/rhc-client-tools-install).

- Install rhc

```bash 
sudo gem install rhc
```

- Configure rhc. 
> *You'll need to have a username and login for [Openshift](http://www.openshift.com) before proceeding.*

```bash 
rhc setup
```

Create an Openshift App and Configure for Virtualenv
---
It is helpful to have a virtualenv on your development machine(s) which matches the environment of your Openshift app. This enables much quicker turnaround time for quick tests of changes, and doesn't require a git commit and push to see the effect of a change.

- Create an Openshift app named Flaskapp.

```bash 
cd ~/src
rhc app create flaskapp python-2.7
```

- Clone App locally into ~/src/flaskapp if you created the app using the web site instead of rhc.

```bash 
rhc git-clone flaskapp
```

- Tell git to ignore the `venv/` dir and other artifacts we are about to create.

```bash 
echo 'venv/' >> ~/src/flaskapp/.gitignore
echo '*.egg-info/' >> ~/src/flaskapp/.gitignore
echo 'dist/' >> ~/src/flaskapp/.gitignore
git commit -am .gitignore 'ignore artifacts'
```

- Setup a venv with the required python version inside `flaskapp`

```bash 
cd ~/src/flaskapp/
# create venv/ dir
virtualenv --python=python2.7 venv
# activate this virtual env
. venv/bin/activate
```

- Install Flask in the new venv we just activated.

```bash 
pip install flask flask-wtf flask-babel markdown flup 
# later we can just just use setup.py
python setup.py install
```

- Tell our app's `setup.py` about our python module requirements.

```bash 
cd ~/src/flaskapp
vim setup.py
```

- Modify the *install_requires* line to look like this:

```python 
install_requires=['Flask','flask-wtf','flask-babel','markdown','flup'],
```

Create Hello World Flask App
---
Create the required directories

```bash 
cd ~/src/flaskapp/wsgi
mkdir -p app/{static,templates}
mkdir tmp
cd ~/src/flaskapp/wsgi/app
```

Create applications files. Pay attention Openshift has some particular requirements.

- *~/src/flaskapp/wsgi/app/\_\_init\_\_.py*

```python 
from flask import Flask  
app = Flask(__name__)  
from app import views
```

- *~/src/flaskapp/wsgi/app/views.py*

```python 
from app import app
@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

- *~/src/flaskapp/wsgi/application* - This application file is required by OpenShift

```python 
#!/usr/bin/python
import os
import sys
sys.path.insert(0, os.path.dirname(__file__) or '.')
PY_DIR = os.path.join(os.environ['OPENSHIFT_HOMEDIR'], "python")
virtenv = PY_DIR + '/virtenv/'
PY_CACHE = os.path.join(virtenv, 'lib', os.environ['OPENSHIFT_PYTHON_VERSION'], 'site-packages')
os.environ['PYTHON_EGG_CACHE'] = os.path.join(PY_CACHE)
virtualenv = os.path.join(virtenv, 'bin/activate_this.py')

try:
    exec(open(virtualenv).read(), dict(__file__=virtualenv))
except IOError:
    pass

from run import app as application
```

 - *~/src/flaskapp/wsgi/run.py* - Called by *application*.

```python 
from app import app
if __name__ == "__main__":
    app.run(debug = True) #We will set debug false in production 
```

Test Flaskapp on localhost
===
Activate venv and run the app.

```bash 
cd ~/src/flaskapp/
. venv/bin/activate
python setup.py install
python run.py
curl http://localhost:5000/index
```

Deploy Flaskapp to Openshift
===
After making local changes, commit them to git, and push them to the origin. Openshift will then automagically install the required flask modules and spin up your Flaskapp.

```bash 
cd ~/src/flaskapp
git add .
git commit -a -m 'Firstsies'
git push
```

Now go check out your new app on [Openshift](http://www.openshift.com)

References
===

* [How-to Install and Configure a Python Flask Dev Environment & Deploy to OpenShift](https://www.openshift.com/blogs/how-to-install-and-configure-a-python-flask-dev-environment-deploy-to-openshift)
* [Virtualenv](http://www.virtualenv.org/en/latest/index.html#what-it-does)
* [Homebrew on Mac OS X](http://hackercodex.com/guide/mac-osx-mavericks-10.9-configuration/)
* [Python Development Environment on Mac OS X](http://hackercodex.com/guide/python-development-environment-on-mac-osx/)
* [Flask-example](https://github.com/openshift/flask-example)
* [Installing RHC Client Tools](https://www.openshift.com/developers/rhc-client-tools-install)

