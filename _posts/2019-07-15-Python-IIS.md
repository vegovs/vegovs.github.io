---
layout: post
title: "Deploying Python Flask app on ISS with pipenv using FastCGI"
author: Vegard Bergsvik Øvstegård
date: 15-07-2019
description: "Deploying Python Flask app on ISS with pipenv using FastCGI"
geometry: margin=2cm
tags: [Python, IIS]
image:
   feature: python.png
---

<details><summary markdown="span"><code>Contents</code></summary>
* TOC
{:toc}
</details>

# Install IIS with FastCGI and python

On your windows server do: 
1. Server Manager -> Manage -> Add Role and Features
	1. Role-based or feature-based installation
	2. Select server 
	3. Choose "Web Server (IIS)"
	4. "Add features"
	5. Accept default until "Role Services.
	6. Add CGI under Application Development 
	7. Install..
2. Web Platform Installer 
	1. Search for "WFastCGI"
	2. Install for preferred Python(3.4).

# Setting up pipenv with your application

1. Set pipenv to install venv in root folders `set PIPENV_VENV_IN_PROJECT="enabled"`
2. Clone or copy your repo to the server 
	* `cd c:\inetpub\wwwroot\`
	* `git clone ...`
3. Install pipenv `pip install --user pipenv`
4. Set up pipenv for the Flask app 
	* `cd yourflaskapp`
	* `pipenv install --python 3.4 -r requirements.txt`
5. Install wfastcgi for the pipenv `pipenv install wfastcgi`
			

# Add a Handler mapping in IIS

1. Copy `C:\inetpub\wwwroot\yourflaskapp\.venv\Lib\site-packages\wfastcgi.py` to the root of your flask application.
2. Add new Handler mapping in IIS
	1. Open "Handler Mappings" in IIS under *yourserver* Home
	2. Click "Add Module Mapping"
		* Request path `*`
		* Module `FastCgiModule`
		* Executable `C:\inetpub\wwwroot\yourflaskapp\.venv\Scripts\pythhon.exe|c:inetpub\wwwroot\yourflaskapp\wfastcgi.py`
		* Name `PythonHandler`
		* Open "Request Restriction" and uncheck "Invoke handler only if requests is mapped to"
		* Click Ok -> Yes
	3. Open "FastCGI Settings" in IIS under *yourserver* Home 
		* Edit the Environment Variables for the handler, select handler -> edit or double click.
			* Double click `...` after selecting Environment Variables 
			* Add `PYTHONPATH` - `c:\inetpub\wwwroot\yourflaskapp\`
			* Add `WSGI_HANDLER` - `app.app` if your flask runner is called app.py

# Add site and edit permissions

* Right click on "Sites" and "Add Website"
* Fill inn a Site name, the path to you Flask app and choose a port. 
* Right click the new site -> Edit permissions
* Go to "Security" -> "Edit" and add the following users with "Full control"
	* IUSR
	* YOURSERVER\IIS_IUSRS


Do a restart of the website and browse it. If you encounter any errors or issues, google it. 
