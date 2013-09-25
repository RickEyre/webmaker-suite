# Webmaker Suite bootstrap/installation script

## Preflight

Make sure you have the following prerequisites installed:

* **Windows only** a basic `cygwin` (http://cygwin.com) or `mingw` (http://mingw.org) installation to ensure you have the `rm`, `git` and `curl` commands available.
* node.js - http://nodejs.org (or through your favourite package manager)
* npm - comes with node.js
* bower - `npm install -g bower` (depends on npm, so install node.js first)
* grunt - `npm install -g grunt-cli` (depends on npm, so install node.js first)
* python - http://python.org (or through your favourite package manager), we depend on 2.7
* pip - instructions on http://www.pip-installer.org/en/latest/installing.html
* mongodb - http://www.mongodb.org/downloads (or through your favourite package manager)
* java - http://www.oracle.com/technetwork/java/javase/downloads/index.html (if not already installed)
* elastic search - http://www.elasticsearch.org (or through your favourite package manager)

**Windows and OSX users take note:**

In addition to having special instructions for installing mongodb and elastic search, Windows and OSX need C++ stacks installed for the `sqlite3` library to compile (used in several tools). The instructions on how to do this can be found in the `OS-specific information` section, further down in this readme. Please follow those instructions before moving on to the next step.

## Steps after preflight

**Step 1**: After your preflight check, clone this repo. If you're on Windows or OSX, make sure you have the C++ stack for your OS installed as per the `specific information` section, further down in this readme.

**Step 2**: open a terminal in the repo's directory and run `node install-webmaker`. This will run a brief bootstrap, after which it will ask for your github username, and S3 credentials (you can leave the S3 credentials blank by simply hitting 'enter' when prompted)

The `node install-webmaker` command can take several runtime options:

1. --username=...
2. --s3key=...
3. --s3secret=...

These do the obvious thing. If you do not use these, and do not have an .env file (which initially you won't), you will be prompted for them during the bootstrap phase, after the `npm install` step for the webmaker-suite package itself finishes.

If you do not provide legal s3 credentials, the tools will fall back to localhost publishing instead. Simply hit enter when prompted for the the s3 key and secret if you don't have, or want to use, real S3 AWS credentials.

4. --skipclone
5. --skipnpm

these will skip the cloning and npm install processes, respectively, in case you need to only run certain parts of the installer.

**Step 3**: Go make some coffee, this'll run for a while (around 15 minutes, but depending on your machine and internet connection, anywhere between 10 and 20 minutes)

After installation completes, the whole suite can be fired up with `node run` (read the next section before running it, though. There are some details you need to know before you type those two words).

Note that this will pipe all output from everything into the same console, so this is great for not-looking-at-the-terminal testing, but not a good idea for doing single-component terminal debugging.

## running the suite

`node run` will run all apps in the webmaker suite. It can take two runtime options:

1. --noes
2. --nomongo
3. --exclude=[comma separated list of names]

The `--exclude` flag takes a comma separated list of app names that should not be started up. If you wish to test only the MakeAPI, for instance, you could issue `node run --exclude=goggles,thimble,popcorn,login,webmaker.org,htmlsanitizer.org`. Note that, for convenience, app names that are of the form "xyz.webmaker.org" can be excluded using only the "xyz" part of the name.

These mostly exist to prevent double-starts for elastic search and mongodb, if you already run these on your machine, since `node run` will try to fire these up for you.

### First run on Windows

When you call `node run` on Windows for the first time, it is entirely possible that you get Windows Firewall notices for java, python, elastic search, mongodb, and Node.js... You'll want to allow all of these on the private network at least (and probably public too, but that's up to you), but these notices WILL interfere with the first run. After allowing everything and the console stops spewing output, hit `ctrl-c` twice, then run `node run` again. There will be no Windows Firewal requests this time round, and the suite should start up without any problems.

## updating the suite

`node update-webmaker` will update all repositories to their current `master` branch. It effectively runs the following commands for each repository:

1. `git fetch mozilla`
2. `git checkout -B master mozilla/master`
3. `git submodule sync`
4. `git submodule update --init --recursive`
5. `rm -rf node_modules`
6. `npm install`
7. `npm cache clean`
8. `npm update`
9. `bower install`
10. `bower update`

## Browser locations

When running the suite, the following locations are available:

* http://localhost:3000 - login.webmaker.org
* http://localhost:3500 - thimble.webmaker.org
* http://localhost:5000 - makeapi
* http://localhost:5050 - htmlsanitizer.org
* http://localhost:7777 - webmaker.org
* http://localhost:8888 - popcorn.webmaker.org
* http://localhost:12416 - goggles.webmaker.org

# OS-specific information

## Webmaker-Suite on Windows

Windows users need to install these two things first (in order):

1. Microsoft Visual Studio C++ 2010 Express, http://go.microsoft.com/?linkid=9709949
2. For Windows 7 x64 (which obviously you're on), the Windows 7 64-bit SDK, http://www.microsoft.com/en-us/download/details.aspx?id=8279

(Without this VC++ stack, the `node-gyp` compiler will crash the `npm install` process when it gets to `sqlite3` and tries to build it without having access to the windows C++ compiler and header files. Since several apps rely on sqlite3 for localhost work, you need these).

Also, if you're on windows, MongoDB and Elastic Search require manual installation, which means putting them in their own folders and extending your PATH:

#### MongoDB on Windows

Download mongodb as a zipfile, create a `MongoDB` folder for it in your program files folder, and copy `mongo.exe`, `mongod.exe` etc. into it. Then add the MongoDB folder to your PATH variable (in windows 7, that's control panel -> system -> advanced system settings -> environment variables -> system variables section).

#### Elastic Search on Windows

Download elastic search, create a folder `C:\elastisearch` (I am not kidding, ES has the worst restrictions), and unpack the .zip package into it. Elastic Search requires Java, so if you don't have that install, grab the most recent JDK and install it. Then symlink the jdk dir as `c:\java`. Again, I am not joking; ES died on spaces anywhere. To do this, open a `cmd` box with administrative rights, to go `C:\` and type:

`C:\> mklink /D java "c:\Program Files (x86)\Java\jdk1.6.0_17"`

(to remove that at some longer point, simply delete the `C:\java` dir. It will unlink, rather than delete your jdk files)

Also add `C:\elasticsearch\bin` to your PATH variable.


## Webmaker-Suite on OSX

You will need the X-Code command line utilities. These are free through Apple's developer appstore. You will also want `brew`, which is a package manager for OSX.

With these, install mongodb and elasticsearch using `brew`, and you should be all set. Convenient!

(If you elect to run these on startup, you will want to make sure to use `node run --noes --nomongo` when you use the suite runner, or `run.js` will try to double-start ES and MongoDB)

### OSX and MongoDB

Mongodb may have told itself to log to a file, instead of to the console. If it has, starting `mongod` will show you a single line of output stating where it's logging to. If this is all you see when you start up Mongo, open its config file by typing the command `vi usr/local/etc/mongod.conf`, and remove the logging instruction in that file (cursor-navigate to an offending line, then press `d` twice to delete it. To save and quite, type `:`, then `wq`, then hit enter). You should now get normal console output when running `mongod`, which is essential for the webmaker-suite to run.
