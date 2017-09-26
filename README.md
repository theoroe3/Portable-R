---
title: "Portable Shiny"
author: "Theo Roe"
date: "26 September 2017"
output: html_document
---

**IMPORTANT :** Remember to remove the shinyApp(ui, server) from end of server file. 

**IMPORTANT :** It is essential that within the server.R file within the server function you have the following code. Note, the server function must contain *session* as an argument. 

```
session$onSessionEnded(function() { 
    stopApp()
    q("no") 
  })
```

This code closes the R terminal as you close the browser. Without this you would have to end the process via ctrl-alt-del.  

**IMPORTANT :** shiny files must be saved as server.R and ui.R, can have other files with them. i.e. functions.R and packages.R


## 1) Editing a previous R Portable folder 
(use the Hello Shiny Portable folder in the repo and ignore the New Portable App folder)

* server, ui etc files go in *app/shiny/*
* *app/config.cfg* contains information and setting for loading. i.e. if you wanted to load packages from somewhere other than CRAN. The appname setting within config.cfg only appears on the box on app load up. 
* *app/packages.txt* is for packages necessary to the shiny app. Comments are placed with # and packages must be placed on seperate lines like so 
```
# this is a comment
package_name_1
package_name_2
# this is another comment
package_name_3
```

* *app/app.R* contains code to launch app, doesn't need to be touched
* the *.bat file* is the file that launches the app. i.e. double click to launch app
* error log for troubleshooting is in *log/error.log*
* *autorun.inf* contains auto launch options. i.e. memory stick name and icon. This saves renaming every memory stick we plan on distributing. Structure shown below.
    + Icon creates the icon for the memory stick and must be placed in the same folder as *autorun.inf* unless a path is specified within *autorun.inf*. Here I have created a jumping rivers icon called *JR.ico*
    + Label creates memory stick name
```
[autorun]
Icon=JR.ico
Label=Jumping Rivers USB App
UseAutoPlay=1
```

## 2) Starting from scratch and more details:
(Use the framework inside the New Portable App folder and ignore the Hello Shiny Portable folder)

### "Install" R-Portable into the framework
The framework can be used with both a system installed version of R or R-Portable.
The latter is recommended as it provides the most application isolation and allows
for applications to be deployed to users unable to install R on their own (i.e.
they lack sufficient system privileges).

Download R-portable from:
https://sourceforge.net/projects/rportable/

Install it into:
```
/<appname/dist/
```

### Customize the framework for your application

#### Install application scripts / assets
For example - a shiny app:

* create a folder called `app/shiny`
* put `ui.R`, `server.R`, and any other related files into `app/shiny`
* edit `app/app.R`:
  
	```r
	# assuming all shiny app code (ui.R and server.R are in ./app/shiny)
	shiny::runApp('./app/shiny')
	```

#### Specify package dependencies
Edit `app/packages.txt` - adding your app's primary package dependencies, one
package per line.

For a `shiny` app that depends on `ggplot`, `app/packages.txt` should look like:
```
jsonlite  # required by DesktopDeployR
shiny
ggplot
```

Packages listed here, along with their dependencies, are installed in a private
application library (`app/library`) when the application is run for the first
time.

Note, the above only works for packages available on CRAN. Custom packages need
to be installed manually.  The recommended method is to use `devtools`:
```
$ Rscript -e "devtools::install('path/to/package', lib = '<appname>/app/library')"
```

#### Configure application launch options
The file `app/config.cfg` is a JSON-ish formatted file that configures how the
application is launched.  Block (`/* ... */`) and line level (`// ...`) comments
are allowed - they are removed to create valid JSON before the file is parsed.

**Root level options:**

| Option     | Description |
| :---       | :--- |
| `appname`  | The name of the application.  Displayed in the title of the progress bar shown during initialization, and used to name folders for logs based on logging settings (see below). |
| `packages` | (Optional: Default: `"http://cran.rstudio.com"`) An object with a single element `cran` that specifies the CRAN mirror to use for installing packages.  Alternatively, this can point to a private CRAN-like package repository for more control over package versions. |
| `r_bindir` | (Optional; Default: `"dist\\R-Portable\\App\\R-Portable\\bin\\"`) A string specifying the path to the `<R_HOME>/bin/` directory for the version of R binaries to use. |
| `logging`  | (Optional; Default: `undefined`) An object with elements specifying logging options (see below) |

**Logging Options:**

| Option            | Description |
| :---              | :--- |
| `filename`        | (Optional; Default: `"error.log"`) Name of the application log file.  This file captures all text sent to `stdout` and `stderr` by the application.  |
| `use_userprofile` | (Optional; Default: `false`) Boolean flag to set where application logs are kept.  If `true` the application log file is written to `%USERPROFILE%/.<appname>/`.  |

#### Rename the application launching batch script file
Rename `appname.bat` as appropriate - e.g. to `MyApplication.bat`.


### Deploy your application
#### Option 1
The entire application folder is copied to the deployed location.  This is the
easiest way to deploy.  Note, it is possible to place / launch an application
from a network share.  If this is the case, be sure to set `logging.use_userprofile: true`
in the application launch configuration.

#### Option 2
Create an installer using NSIS installer or InnoSetup.  This is ideal for installing
on isolated workstations.  Both R-portable and package dependencies can be compiled
with the installer.  This means that a "first run" that ensures all package
dependencies, must occur before compiling an installer.

**TBD**: an example InnoSetup installer compile script


## Notes

### Using a different (newer) version of R
Either replace `./dist/R-Portable/` with the version of `R-Portable` that is
required or modify `./app/config.cfg` to point to the desired R installation
(e.g. a system install).


### Version tracking
Due to their potentially large sizes, it is not recommended that the following
folders be tracked by version control (i.e. Git):

* `/app/library/`
* `/dist/R-Portable`

### Application Structure
```
/<appname>          # - application deployment root
./app/              # - application working directory

	./library/        # - application specific package library

	./shiny/          # - application framework folder (in this case, shiny)
		./global.R      # - global constants and functions for shiny-app
		./server.R      # - server processing function for shiny-app
		./ui.R          # - user interface definition function for shiny-app

	./app.R           # - application launch entry script
	./config.cfg      # - application configuration file
	./packages.txt    # - list of primary package dependencies
	./...             # - other application files

./dist/             # - application launch framework
	./R-Portable/     # - "vanilla" R interpreter (downloaded separately)
	./script/
		./R
			./run.R       # - R environment initialization and application launch
		./wsf
			./js
				./JSON.minify.js  # - JS to JSON minifier, allows comments in JSON files
				./json2.js  # - JSON parsing library
				./run.js    # - OS application launch script
			./run.wsf     # - merges javascript dependencies and launch script
	./USAGE.md        # - notes on how the dist folder is structured

/<appname>.bat      # - batch file to start application
/README.md          # - this file or brief description of application
```
