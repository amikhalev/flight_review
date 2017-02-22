### Flight Review ###

This is a web application for flight log analysis. It allows users to upload
ULog flight logs, and analyze them through the browser.

It uses the [bokeh](http://bokeh.pydata.org) library for plotting and the
[Tornado Web Server](http://www.tornadoweb.org).


#### Installation and Setup ####

- clone the repository
- use python3
- `pip3 install bokeh jinja2 pyulog simplekml` (at least version 0.12.4 of bokeh is
  required)
- `sudo apt-get install sqlite3`
- configure web server config (this can be skipped for a local installation):
  create a file `config_user.ini` and copy and adjust the sections and values
  from `config_default.ini` that should be overridden.
- `./setup_db.py` to initialize the database


#### Usage ####

For local usage, the server can be started directly with a log file name,
without having to upload it first:
```
./serve.py -f <file.ulg>
```

The `plot_app` directory contains a bokeh server application for plotting. It
can be run stand-alone with `bokeh serve --show plot_app` (or with `cd plot_app;
bokeh serve --show main.py`, to start without the html template).

The whole web application is run with the `serve.py` script. Run `./serve.py -h`
for further details.


#### Interactive Usage ####
The plotting can also be used interative using a Jupyter Notebook. It
requires python knowledge, but provides full control over what and how to plot
with immediate feedback.

- `pip3 install jupyter`
- Start the notebook: `jupyter notebook`
- open the `testing_notebook.ipynb` file

### TODO list ###
- Google Maps seems to have some problems (initialization, scaling and zooming
  issues). This is bokeh
- maximum upload size is currently limited to 100MB. This requires a bokeh
  setting.
- add SSL
- Not all bokeh widgets seem to be responsive to size changes
- better downsampling (use JS callback with queue & timeout (like
  InteractiveImage) and better algorithm)
- add location obfuscation option
- user management: login, per-user display templates
- download CSV option?
- ...


### Implementation ###
Reading ULog files is expensive and thus should be avoided if not really
necessary. There are two mechanisms helping with that:
- Loaded ULog files are kept in RAM using an LRU cache with configurable size
  (when using the helper method). This works from different requests and
  sessions and from all source contexts.
- There's a LogsGenerated DB table, which contains extracted data from ULog
  for faster access.

Tornado uses a single-threaded event loop. This means all operations should be
non-blocking (see also http://www.tornadoweb.org/en/stable/guide/async.html).


#### Notes about python imports ####
Bokeh uses dynamic code loading and the `plot_app/main.py` gets loaded on each
session (page load) to isolate requests. This also means we cannot use relative
imports. We have to use `sys.path.append` to include modules in `plot_app` from
the root directory (Eg `tornado_handlers.py`). Then to make sure the same module
is only loaded once, we use `import xy` instead of `import plot_app.xy`.
It's useful to look at `print('\n'.join(sys.modules.keys()))` to check this.


#### Contributing ####
Contributions are welcome! Just open a pull request with detailed description
why the changes are needed, or open an issue for bugs, feature requests, etc...

