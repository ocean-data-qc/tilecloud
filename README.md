TileCloud
=========

One tile to rule them all



Introduction
============

TileCloud is a powerful utility for generating, managing, transforming, visualising and map tiles in multiple formats.  It can create, read update, delete tiles in multiple back ends, called TileStores. Existing TileStores include:

* HTTP/REST in any layout

* WMTS

* Amazon S3

* [MBTiles](https://github.com/mapbox/mbtiles-spec)

* [TileJSON](https://github.com/mapbox/TileJSON)

* Local file system

* Log files in any format

TileCloud is not limited to image tiles, it can also handle other tile data such as [UTFGrid](https://github.com/mapbox/utfgrid-spec), or elevation data in JSON format.

TileCloud uses Python's generators and iterators to efficiently stream tens of millions of tiles, and can handle multiple tiles in parallel using Python's [multiprocessing](http://docs.python.org/library/multiprocessing.html) library.

Example tasks that TileCloud makes easy include:

* Visualize tiles stored in any TileStore with [OpenLayers](http://www.openlayers.org/), [Google Maps](http://maps.google.com/), [Leaflet](http://leaflet.cloudmade.com/), [Polymaps](http://polymaps.org/), [Modest Maps](http://www.modestmaps.com/), and [OpenWebGlobe](http://www.openwebglobe.org/).

* Convert sixty million PNG tiles stored in S3 to JPEG format with different quality settings at different zoom levels.

* Transform image formats and perform arbitrary image transformations on the fly, including PNG optimization.

* Generate semi-transparent tiles with embedded tile coordinates for debugging.

* Pack multiple tile layers into a single tile on the server.

* Efficiently calculate bounding boxes and detect missing tiles in existing tile datasets.

* Simulate fast and slow tile servers.

* Efficiently delete millions of tiles in S3.

* Read JSON tiles from a tarball, compress them, and upload them.



Getting started
===============

TileCloud depends on [Bottle](http://bottlepy.org/), [pyproj](http://code.google.com/p/pyproj/) and [Pycairo](http://cairographics.org/pycairo/).  It's easiest to install them with `pip` in a `virtualenv`:

	$ virtualenv .
	$ . bin/activate
	$ pip install bottle pyproj

Unfortunately, Pycairo does not install with `pip`, so use your system's package manager to install the system package (on Ubuntu it's `python-cairo`).

For a quick demo, run

	$ ./tc-viewer --root=3/4/2 'http://gsp2.apple.com/tile?api=1&style=slideshow&layers=default&lang=en_GB&z=%(z)d&x=%(x)d&y=%(y)d&v=9'

and point your browser at <http://localhost:8080/>.  Type `Ctrl-C` to terminate `tc-viewer`.

Next, download an example MBTiles file from [MapBox](http://mapbox.com/), such as [Geography Class](http://tiles.mapbox.com/mapbox/map/geography-class).  We can quickly find out more about this tile set with the `tc-info` command:

	$ ./tc-info -t count geography-class.mbtiles
	87381

	$ ./tc-info -t bounding-pyramid -r geography-class.mbtiles
	0/0/0:+1/+1
	1/0/0:+2/+2
	2/0/0:+4/+4
	3/0/0:+8/+8
	4/0/0:+16/+16
	5/0/0:+32/+32
	6/0/0:+64/+64
	7/0/0:+128/+128
	8/0/0:+256/+256

Now, display this MBTiles tile set on top of the OpenStreetMap tiles and a debug tile layer:

	$ ./tc-viewer tiles.openstreetmap_org geography-class.mbtiles tiles.debug.black

You'll need to point your browser at <http://localhost:8080/> and choose your favourite library.

`tc-info` and `tc-viewer` are utility programs.  Normally you use TileCloud by writing short Python programs that connect the TileCloud's modules to perform the action that you want.

As a first example, run the following:

        $ PYTHONPATH=. examples/download.py

This will download a few tiles from [OpenStreetMap](http://www.openstreetmap.org/) and save them in a local MBTiles file called `local.mbtiles`.  Look at the source code to `examples/download.py` to see how it works.  If there are problems with the download, just interrupt it with `Ctrl-C` and re-run it: the program will automatically resume where it left off.

Once you have downloaded a few tiles, you can view them directly with `tc-viewer`:

	$ ./tc-viewer --root=4/8/5 local.mbtiles tiles.debug.black

Point your browser at <http://localhost:8080> as usual.  The `--root` option to `tc-viewer` instructs the viewer to start at a defined tile, rather than at 0/0/0, so you don't have to zoom in to find the tiles that you downloaded.



Using TileCloud as a cheap-and-cheerful tile server
===================================================

`tc-viewer` can be used as a lightweight tile server, which can be useful for development, debugging and off-line demos.  The TileStores passed as arguments to `tc-viewer` are available at the URL:

	http://localhost:8080/tiles/{index}/tiles/{z}/{x}/{y}

where `{index}` is the index of the TileStore on the command line (starting from 0 for the first tile store), and `{z}`, `{x}` and `{y}` are the components of the tile coordinate.  The second `tiles` in the URL is present to work around assumptions made by OpenWebGlobe.  This layout is directly usable by most mapping libraries, see the code in `views/*.tpl` for examples.  The host and port can be set with the `--host` and `--port` command line options, respectively.

Note that there is no file extension.  `tc-viewer` will automatically set the correct content type and content encoding headers if it can determine them, and, failing this, most browsers will figure it out for themselves.

For convenience, `tc-viewer` serves everything in the `static` directory under the URL `/static`.  This can be used to serve your favourite mapping library and/or application code directly for testing purposes.

By default, `tc-viewer` will use [Tornado](http://www.tornadoweb.org/) as a web server, if it is available, otherwise it will fall back to [WSGIRef](http://docs.python.org/library/wsgiref.html).  Tornado has reasonably good performance, and is adequate for local development and off-line demos, especially when used with a MBTiles TileStore.  WSGIRef has very poor performance (it handles only one request at a time) and as such can be used as a "slow" tile server, ideal for debugging tile loading code or testing how your web application performs over a slow network connection.  `tc-viewer` is particularly slow when used to proxy tiles being served by a remote server.  You can set the server explicitly with the `--server` option.

`tc-viewer` sets the `Access-Control-Allow-Origin` header to `*` for all the tiles it serves, this allows the tiles to be used as textures for WebGL applications running on different hosts/ports.  For more information, see [Cross-Domain Textures](https://developer.mozilla.org/en/WebGL/Cross-Domain_Textures).

`tc-viewer` is designed as a development tool, and the power that it offers comes at the expense of fragility.  It makes many assumptions - including the benevolence of the user - that make it entirely unsuitable as a generic tile server.  It should only be used in development or demonstration environments.



Contributing
============

Please report bugs and feature requests using the [GitHub issue tracker](https://github.com/twpayne/tilecloud/issues).

For pull requests, it is very much appreciated if your code passes [pep8](http://pypi.python.org/pypi/pep8) and [pyflakes](http://pypi.python.org/pypi/pyflakes) without warnings, with the exception of pep8 warning "E501 line too long", which is allowed.  You can run pep8 and pyflakes on the codebase with the command:

	$ make pep8 pyflakes



License
=======

Copyright (c) 2012, Tom Payne <twpayne@gmail.com>
All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

vim: set filetype=markdown spell spelllang=en textwidth=0: