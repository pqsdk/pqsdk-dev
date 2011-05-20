---
categories: titanium appcelerator, python, webkit, javascript
date: 2011/03/11 18:00:00
title: Building a Titanium Appcelerator desktop app using Python
---

I have never been impressed by desktop applications on Windows that look like they were built with MFC or VB forms or .NET forms with countless number of controls and controls.  On the hand, there are these desktop applications (usually from anti-virus software like McAfee) that are sometimes windowless and often sport stunning user interface (UI).  These UI look like a web page with nice colors, nice layout and pleasant-looking fonts.  I never knew even where to start building something like that in an efficient way.  Certainly, not with any tool by Microsoft.  [U++ framework](http://www.ultimatepp.org/index.html) does not have anything either.  But when [QT had this Webkit Integration](http://qt.nokia.com/products), it started to look possible.  The task of building a useful desktop application remains quite not so easy.  Unless you put together a sample tutorial that points the webkit to [google.com](http://google.com), you would have to build a service that also embed a small, lightweight web server, where your webkit would point to.  It can start to get, not impossible, but quite involved.

The company [Appcelerator](http://www.appcelerator.com/) released this product called [Titanium Desktop](http://www.appcelerator.com/products/titanium-desktop-application-development/) and [Titanium Mobile](http://www.appcelerator.com/products/titanium-mobile-application-development/) that I think are just brilliant.  My interest is with the mobile product, but I'll start experimenting with the desktop.  Follow the 2 above links, you can start to see how appealing they are.  What drew me first to Titanium is when I read that it has full support for Python, Javascript, HTML, CSS.  It can also do Ruby and PHP, but those are moot points for me since I just started learning Ruby and have not had much experience with it anyway.  At first look, there seems to be good online documentation, an online Developer Center, a forum, etc.  The problem is there not much in any one of those places to read and learn from.  In my case, I want to use Javascript and Python to build my application.  At the time of this writing, there just doesn't seem to be much:

[http://developer.appcelerator.com/doc/desktop/python](http://developer.appcelerator.com/doc/desktop/python).</p>
    
[http://developer.appcelerator.com/question/84901/very-basic-example-with-python](http://developer.appcelerator.com/question/84901/very-basic-example-with-python).
    
[http://developer.appcelerator.com/questions/tag/python](http://developer.appcelerator.com/questions/tag/python)
    
[http://www.brighthub.com/hubfolio/matthew-casperson/articles/52383.aspx](http://www.brighthub.com/hubfolio/matthew-casperson/articles/52383.aspx).  This is a good article, but it does not address how to call Python.
    
[http://stackoverflow.com/questions/4026814/appcelerator-titanium-developer-with-python-php-ruby](http://stackoverflow.com/questions/4026814/appcelerator-titanium-developer-with-python-php-ruby).  Even on StackOverflow, there's not much there.
    
Of course, I was completely lost.  Some of my initial questions are:

1. Do I need to embed some kind of Python web server?
    
2. Can I , must I, should I use Django or Pylons or Flask?
    
3. How can I call Python code? By making HTTP request? but to which URL end point and how to specify them?
    
4. Maybe call Python directly? But what's the syntax? How to return data from Python code? Where to put Python code?

There's a [kitchen sink](https://github.com/appcelerator/KitchenSink) sample.  I browsed briefly, but did not import it, and didn't see any Python code that could help me getting started.  So after struggling for a few days, I now have an application that is built using Python and I want to share.  Once I look back nopw at the result, it looks quite simple.  But that didn't seem too obvious to me at first.

Normally, I would use my Ubuntu laptop, but for now, there's still an issue with Ubuntu that I think Appcelerator must fix sooner than later.  Or else, it would defeat their purpose of cross-platform.  So if you want to follow along, please use your Windows machine.  (I will try on OS X later and keep posted).

First, go ahead and launch the Titanium app:

![Create new Project](https://github.com/pragmaticobjects/pqsdk-blog-img/raw/master/Building%20a%20Titanium%20Appcelerator%20desktop%20app%20using%20Python/TitaniumFileExplorer%20New%20Project.png "Create new Project")

What you put in here is self-explanatory.  Once you select "Create Project", Titanium creates a directory structure similar to the next picture:

![TitaniumFileExplorer directory structure](https://github.com/pragmaticobjects/pqsdk-blog-img/raw/master/Building%20a%20Titanium%20Appcelerator%20desktop%20app%20using%20Python/TitaniumFileExplorer%20Directory%20Structure.png "TitaniumFileExplorer directory structure")

Initially, Titanium only generates the "index.html" file in the "Resources" directory.  We would have to create the "js", "css", "img" sub directories just like we would when developing web applications.  Since I want to code in Python, I create a Python source file called "fileexplorer.py", and also put it in this "Resources" directory.  In the "TitaniumFileExplorer" directory, there's also a file called "tiapp.xml" (not shown in the picture) where we can specify attributes like width, height, max width, max height, title, full screen, etc.

The ["fileexplorer.py"](https://gist.github.com/867091) code is listed below:

$$code(lang=bash)  
import sys, os, time

def listFiles(val):
	curdir = os.getcwd()
	files = []
	for filename in os.listdir(curdir):
		if (not filename.startswith('.')):
			s = os.stat(os.path.join(curdir, filename))
			lastmodified = time.strftime("%Y-%m-%d %I:%M %p", time.localtime(s.st_mtime))
			size = os.path.getsize(os.path.join(curdir, filename))
			isdir = os.path.isdir(os.path.join(curdir, filename))
            if isdir is True:
				filesize = ""
			else:
				if size < 1000:
					filesize = "1 KB"
				elif size < 1000000:
					filesize = "%.2f %s" % ((size / 1000), ' KB')
				else:
					filesize = "%.2f %s" % ((size / 1000000), 'MB')
			fileattr = {
			'filename' : filename,
			'isdir': isdir,
			'filesize': filesize,
			'lastmodified': lastmodified
			}
			files.append(fileattr)
	return files
$$/code

Inside the function fileFiles(), I made some os file-related calls.  The only thing to notice here is the function returns a JSON object called **files**.  Next, we will see how this function file gets invoked from Javascript (in [**index.html**](https://gist.github.com/867096)).

$$code(lang=javascript) 
<script>
	$(document).ready(function() {
		jQuery("#files-list").jqGrid({
			datatype: "local",
			height: 350,
			colNames:['File Name', 'Type', 'Size', 'Last Modified'],
			colModel:[
				{name: 'filename', index: 'filename', width: 400, sorttype: "string"},
				{name: 'isfolder', index: 'isfolder', width: 110, sorttype: "string"},
				{name: 'filesize', index: 'filesize', width: 100, sorttype: "float"},
				{name: 'lastmodified', index: 'lastmodified', width: 320, sorttype: "string"}
			],
			multiselect: true,
			caption: "Titanium File Explorer"
		});
		var files = fileexplorer.listFiles('');
		$.each(files, function(i, e) {
			var rowData = {
				filename: e.filename,
				isfolder: (e.isdir == true) ? 'Folder': 'File',
				filesize: e.filesize,
				lastmodified: e.lastmodified
			};
			jQuery("#files-list").jqGrid('addRowData', i+1, rowData);
		});
	});
</script>
$$/code

For this simple/demo application, I decide to use jQuery and [jqGrid](http://www.trirand.com/blog/).  If you havealready taken a look at the HTML file, you would see there's a bunch of Javascript files that get included, which should not be too surprising.  Everything just look like a web app.  The only difference is the inclusion of Python code:

$$code(lang=python) 
<script type="text/python">
    import fileexplorer
</script>
$$/code

In the document ready handler, after some initialization of the grid, the call to the above Python function is made.  This is where things are different from a web app, where you are expected to make an Ajax request/call here.  When the call to Python finishes, as it returns a JSON list (or array) of objects, I simply iterate and add each element to a row in the grid.  Again this is different from a web app.  In a web app code, you would iterate and process whatever inside a callback function.  Other than these key differences, the code is quite simple:

$$code(lang=javascript)
var files = fileexplorer.listFiles(''); 
$.each(files, function(i, e) {
    var rowData = {
        filename: e.filename,
        isfolder: (e.isdir == true) ? 'Folder': 'File',
        filesize: e.filesize,
        lastmodified: e.lastmodified
    };
    jQuery("#files-list").jqGrid('addRowData', i+1, rowData);
});
$$/code

**Final Thoughts:**

    

 

            



