# TouchDesigner Process Management
A look at how you can launch another process from TouchDesigner with Python, and how you can kill that process.

## Contributing Programers / Artists ##
**Matthew Ragan** |[ matthewragan.com ](http://matthewragan.com)  

## TouchDesigner Version ##
All work in this repo is being done in TouchDesigner 099. 

## OS Limitation
This particular approach works for sure in Windows, but is currently untested on MacOS.

## Overview
At some point you'll need to split up the work of a single project into multiple processes. This happens for lots of reasons - maybe you want to break your control interface out from your output elements, or maybe you want to start up another tool you've built - you name it, there are lots of reasons you might want to launch another process, and if you haven't found a reason to you... chances are you will soon. 

The good news is that we can do this with a little bit of python. We need to import a few extra libraries, and we need to do a little leg work - but once we get a handle on those things we have a straightforward process on our hands.

### Getting Started
First things first, start by downloading or cloning the whole repo. We'll start by opening the `process-management.toe` file. You might imagine that this is the toe file that you're launching processes from, or you might think of this as your main control toe file. You'll notice that there's also a toe file called `other-app.toe`. This is the file we're going to launch from within TouchDesigner. At this point feel free to open up that file - you'll see that it starts in perform mode and says that it's some other process. Perfect. You should also notice that it says "my role is," but nothing else. Don't worry, it's this way on purpose. 

### Process-management.toe
In this toe file you'll see three buttons:
* Launch Process
* Quit Process
* Quit Process ID None

#### Launch Process
This button will run the script in `text_start_process`.

```python
import os
import subprocess

# we need to know the location of our app
app                     = "{}/TouchDesigner099.exe".format(app.binFolder)

# we also need to know the location of our file
file                    = "{}/other-process.toe".format(project.folder)

# we're going to set an environment variable for fun
os.environ['ROLE']      = "render1"

# we can start the process with a Popen() call
app_process             = subprocess.Popen([app, file])

# next we can find our process ID
app_id                  = app_process.pid

# to make this a little easier to quit our process we'll 
# put some of these things into storage for later access
other_app       = {
    "app_process"       : app_process,
    "app_id"            : app_id
}

parent().store('other_app', other_app)
```

So, what's happening here? First we need to import a few other libraries that we're going to use - os and subprocess. From there we need to identify the application we're going to use, and the file we're going to launch. Said another way we want to know what program we're going to open our toe file with. You'll see that we're doing something a little tricksy here. In Touch, the all class has a member called `binFolder` - this tells us the location of the Touch Binary files, which happens to include our executable file. Rather than hard coding a path to our binary we can instead use the path provided by Touch - this has lots of advantages and should mean that your code is less likely to break from machine to machine. 

Similarly, you'll see that we location our `other-process.toe` by looking at our project folder (where `process-management.toe` is saved) and locating our `other-process.toe` file relative to our main project file. 

So far so good. You should also see that we're setting an environment variable with `os.environ`. This is an interesting place where we can actually set variables for a Touch process at start. Why do this? Well, you may find that you have a single toe file that you want to configure differently for any number of reasons. If you're using a single toe file configuration, you might want to launch your main file to default as a controller, and a another instance of the same file in an output configuration, and maybe another instance of the same app to handle some other process. If that's not interesting to you, you can comment out that line - but it might at least be worth thinking about before you add that pound sign.

Next we use a subprocess.Popen() call to start our process - by providing the all, and the file as arguments in a list. We can also grab our process ID (we'll use that later) while we're here.

Finally we'll build a little dictionary of our attributes and put that all in storage. I'm using a dictionary in this example since you might find that you need to launch multiple instances, and having a nice way to keep them separate is handy. 

Okay. To see this work, let's make that button viewer active and click it - tada! At this point you should see another TouchDesigner process launch and this time around the name that was entered for our `ROLE` environment variable shows up in our second touch process: "some other process, my role is render1"

Good to know is that if you try to save your file now you'll get an error. That's because we've put a subprocess object into storage and it can't persist between closing and opening. Your file will be saved, but our little dictionary in storage will be lost.

#### Quit Process
This little button kills our `other-app.toe` instance.

```python
# first we need to grab our stored details
other_app = parent().fetch('other_app')

# next we can kill the process
other_app['app_process'].kill()

# finally we'll unstore everything so we don't
# see any errors when we save
parent().unstore("*")
```

Much simpler than our last script this one first grabs our dictionary that's in storage, grabs the subprocess object, and then kills it. Next we unstore all of the bits in storage so we can save our toe file without any warning messages.

#### Quit Process ID
Okay - so what happens if you want to just kill a process by it's ID, not by storing the whole subprocess object? You can do that.

```python
import os
import signal

# first we need to find our process ID
# which we can grab from the dictionary 
# we put into storage
pid = parent().fetch('other_app')['app_id']

# next we can kill this process based 
# only on its process id
os.kill(pid, signal.SIGTERM)

parent().unstore("*")
```

In this case we can use the os module to issue a kill call with our process id. If we look at the [os documentation](https://docs.python.org/2.6/library/os.html) we'll see that we need a pid and a sig - which is why we're also importing signal. 


### Take Aways
This may or may not be useful in your current work flow, but it's handy to know that there are ways to launch and quit another toe file. Better yet, this same idea doesn't have to be limited to use with Touch. You might use this to lunch or control any other application your heart desires. 