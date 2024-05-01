# Installing Beancount (v2)<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca) - Updated: June 2015

[<u>http://furius.ca/beancount/doc/install</u>](http://furius.ca/beancount/doc/install)

*Instructions for downloading and installing Beancount on your computer.*

<table><tbody><tr class="odd"><td><em>This document is about Beancount v2; Beancount v3 is in development and uses a completely different build and installation system. For instructions on building v3, see <a href="installing_beancount_v3.md"><u>this document</u></a>.</em></td></tr></tbody></table>

## Releases<a id="releases"></a>

Beancount is a mature project: the first version was written in 2008. The current rewrite of Beancount is stable. (Technically, this is what I call version 2.x beta).

I’m still working on this Beancount code every weekend these days, so it is very much in active development and evolving, though the great majority of the basic features are basically unchanging. I’ve built an extensive suite of tests so you can consider the “default” branch of the repository as stable. New features are developed in branches and only merged in the “default” branch when fully stable (the entire battery of tests passes without failures). Changes to “default” are posted to the [<u>CHANGES</u>](https://github.com/beancount/beancount/tree/v2/CHANGES) file and a corresponding email is sent to the [<u>mailing-list</u>](https://groups.google.com/forum/#!forum/beancount).

There's a [<u>PyPI</u>](https://pypi.python.org/pypi/beancount) page.

## Where to Get It<a id="where-to-get-it"></a>

This is the official location for the source code:

> [<u>https://github.com/beancount/beancount</u>](https://github.com/beancount/beancount)

Download it like this, by using Git to make a clone on your machine:

    git clone https://github.com/beancount/beancount

## How to Install<a id="how-to-install"></a>

### Installing Python<a id="installing-python"></a>

Beancount uses Python 3.5[^1] or above, which is a pretty recent version of Python (as of this writing), and a few common library dependencies. I try to minimize dependencies, but you do have to install a few. This is very easy.

First, you should have a working Python install. Install the latest stable version &gt;=3.5 using the download from [<u>python.org</u>](http://python.org). Make sure you have the development headers and libraries installed as well (e.g., the “Python.h” header file). For example, on a Debian/Ubuntu system you would install the **`python3-dev`** package.

Beancount supports setuptools since Feb 2016, and you will need to install dependencies. You will want to have the “pip3” tool installed. It’s probably installed by default along with Python3—test this out by invoking “pip3” command. In any case, under a Debian/Ubuntu system you would simply install the **`python3-pip`** package.

#### Python Dependencies<a id="python-dependencies"></a>

Note that in order to build a full working Python install from source, you will probably need to install a host of other development libraries and their corresponding header files, e.g., libxml2, libxslt1, libgdbm, libmp, libssl, etc. Installing those is dependent on your particular distribution and/or OS. Just make sure that your Python installation has all the basic modules compiled for its default configuration.

### Installing Beancount using pip<a id="installing-beancount-using-pip"></a>

This is the easiest way to install Beancount. You just install Beancount using

    sudo -H python3 -m pip install beancount

This should automatically download and install all the dependencies.

Note, however, that this will install the latest version that was pushed to the [<u>PyPI repository</u>](https://pypi.python.org/pypi/beancount/) and not the very latest version available from source. Releases to PyPI are made sporadically but frequently enough not to be too far behind. Consult the [<u>CHANGES file</u>](https://github.com/beancount/beancount/tree/v2/CHANGES) if you’d like to find out what is not included since the release date.

### Installing Beancount using pip from Repository<a id="installing-beancount-using-pip-from-repository"></a>

You can also use pip to install Beancount from its source code repository directly:

    sudo -H python3 -m pip install git+https://github.com/beancount/beancount#egg=beancount

### Installing Beancount from Source<a id="installing-beancount-from-source"></a>

Installing from source offers the advantage of providing you with the very latest version of the stable branch (“default”). The default branch is as stable as the released version.

#### Obtain the Source Code<a id="obtain-the-source-code"></a>

Get the source code from the official repository:

    git clone https://github.com/beancount/beancount

#### Install third-party dependencies<a id="install-third-party-dependencies"></a>

You might need to install some non-Python library dependencies, such as libxml2-dev, libxslt1-dev, and perhaps a few more. Try to build, it should be obvious what’s missing. If on Ubuntu, use apt-get.

If installing on Windows, see the Windows section below.

#### Install Beancount from source using pip3<a id="install-beancount-from-source-using-pip3"></a>

You can then install all the dependencies and Beancount itself using pip:

    cd beancount
    sudo -H python3 -m pip install .

#### Install Beancount from source using setup.py<a id="install-beancount-from-source-using-setup.py"></a>

First you’ll need to install dependent libraries. You can do this using pip:

    sudo -H python3 -m pip install python-dateutil bottle ply lxml python-magic beautifulsoup4

Or equivalently, you may be able to do that using your distribution, e.g., on Ubuntu/Debian:

    sudo apt-get install python3-dateutil python3-bottle python3-ply python3-lxml python3-bs4 …

Then, you can install the package in your Python library using the usual setup.py invocation:

    cd beancount
    sudo python3 setup.py install

Or you can install the package in your user-local Python library using this:

    sudo python3 setup.py install --user

Remember to add `~/.local/bin` to your path to access the local install.

#### Installing for Development<a id="installing-for-development"></a>

If you want to execute the source in-place for making changes to it, you can use the setuptools “develop” command to point to it:

    cd beancount
    sudo python3 setup.py develop 

Warning: This modifies a .pth file in your Python installation to point to the path to your clone. You may or may not want this.

The way I work myself is *old-school*; I build it locally and setup my environment to find its libraries. You build it like this:

    cd beancount
    python3 setup.py build_ext -i

Finally, both the PATH and PYTHONPATH environment variables need to be updated for it:

    export PATH=$PATH:/path/to/beancount/bin
    export PYTHONPATH=$PYTHONPATH:/path/to/beancount

### Installing from Packages<a id="installing-from-packages"></a>

Various distributions may package Beancount. Here are links to those known to exist:

-   Arch: [<u>https://aur.archlinux.org/packages/beancount/</u>](https://aur.archlinux.org/packages/beancount/)

### Windows Installation<a id="windows-installation"></a>

#### Native<a id="native"></a>

Installing this package by pip requires compiling some C++ code during the installation procedure which is only possible if an appropriate compiler is available on the computer, otherwise you will receive an error message about missing *vsvarsall.bat* or *cl.exe*.

To be able to compile C++ code for Python you will need to install the same major version of the C++ compiler as your Python installation was compiled with. By running *python* in a console and looking for a text similar to *\[MSC v.1900 64 bit (AMD64)\]* you can determine which compiler was used for your particular Python distribution. In this example it is *v.1900*.

Using this number find the required Visual C++ version [<u>here</u>](https://stackoverflow.com/questions/2676763/what-version-of-visual-studio-is-python-on-my-computer-compiled-with). Since different versions seem to be compatible as long as the first two digits are the same you can in theory use any Visual C++ compiler between 1900 and 1999.

According to my experience both Python 3.5 and 3.6 was compiled with MSC v.1900 so you can do either of the following to satisfy this requirement:

-   Install the standalone [<u>Build Tools for Visual Studio 2017</u>](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=15) or

-   Install the standalone [<u>Visual C++ Build Tools 2015</u>](http://go.microsoft.com/fwlink/?LinkId=691126) or

-   Modify an existing Visual Studio 2017 installation

    -   Start the Visual Studio 2017 installer from *Add or remove programs*

    -   Select *Individual components*

    -   Check *VC++ 2017 version 15.9 v14.16 latest v141 tools* or newer under *Compilers, build tools, and runtimes*

    -   Install

-   Visual Studio 2019

    -   add C++ build tools: C++ core features, MSVC v142 build tools

If cl.exe is not in your path after installation, run Developer Command Prompt for Visual Studio and run the commands there.

#### With Cygwin<a id="with-cygwin"></a>

Windows installation is, of course, a bit different. It’s a breeze if you use Cygwin. You just have to prep your machine first. Here’s how.

-   Install the latest [<u>Cygwin</u>](https://www.cygwin.com/). This may take a while (it downloads a lot of stuff), but it is well worth it in any case. But before you kick off the install, make sure the following packages are all manually enabled in the interface provided by setup.exe (they’re not selected by default):

    -   python3

    -   python3-devel

    -   python3-setuptools

    -   git

    -   make

    -   gcc-core

    -   flex

    -   bison

    -   lxml

    -   ply

-   Start a new Cygwin bash shell (there should be a new icon on your desktop) and install the pip3 installer tool by running this command:  
    **easy\_install-3.4 pip  
    **Make sure you invoke the version of easy\_install which matches your Python version, e.g. easy\_install-3.5 if you have Python 3.5 installed, or more.

At this point, you should be able to follow the instructions from the previous sections as is, starting from “Install Beancount using pip”.

#### With WSL<a id="with-wsl"></a>

The newly released Windows 10 Anniversary Update brings WSL 'Windows Subsystem for Linux' with [<u>bash on Ubuntu on Windows</u>](https://blogs.msdn.microsoft.com/wsl/2016/07/08/bash-on-ubuntu-on-windows-10-anniversary-update/) (installation instructions and more at [<u>https://msdn.microsoft.com/commandline/wsl/about</u>](https://msdn.microsoft.com/commandline/wsl/about)).

This makes beancount installation easy, from bash:

    sudo apt-get install python3-pip
    sudo apt-get install python3-lxml
    sudo pip3 install m3-cdecimal
    sudo pip3 install beancount --pre

This is not totally “Windows compatible”, as it is running in a pico-process, but provides a convenient way to get the Linux command-line experience on Windows. *(Contrib: willwray)*

#### Notes on lxml<a id="notes-on-lxml"></a>

Some users have reported problems installing lxml, and a solution: when installing lxml with pip (under Cygwin), using this may help:

    STATIC_DEPS=true pip install lxml

### Checking your Install<a id="checking-your-install"></a>

You should be able to run the binaries from [<u>this document</u>](running_beancount_and_generating_reports.md). For example, running bean-check should produce something like this:

    $ bean-check

    usage: bean-check [-h] [-v] filename
    bean-check: error: the following arguments are required: filename

If this works, you can now go to the [<u>tutorial</u>](tutorial_example.md) and begin learning how Beancount works.

### Reporting Problems<a id="reporting-problems"></a>

If you need to report a problem, either send email on the mailing-list or [<u>file a ticket</u>](https://github.com/beancount/beancount/issues) on Github. Running the following command lists the presence and versions of dependencies installed on your computer and it might be useful to include the output of this command in your bug report:

    python3 -m beancount.scripts.deps

## Editor Support<a id="editor-support"></a>

There is support for some editors available:

-   Emacs support is provided [<u>in a separate repo</u>](https://github.com/beancount/beancount-mode). See the [<u>Getting Started</u>](getting_started_with_beancount.md) text for installation instruction.

-   Support for [<u>editing with Sublime</u>](https://sublime.wbond.net/packages/Beancount) has been contributed by [<u>Martin Andreas Andersen</u>](https://groups.google.com/d/msg/beancount/WvlhcCjNl-Q/s4wOBQnRVxYJ). See [<u>his github repo</u>](https://github.com/draug3n/sublime-beancount).

-   Support for editing with Vim has been contributed by [<u>Nathan Grigg</u>](https://github.com/nathangrigg). See [<u>his GitHub repo</u>](https://github.com/nathangrigg/vim-beancount).

-   Visual Studio Code currently has two extensions available. Both have been tested on Linux.

    -   [<u>Beancount</u>](https://marketplace.visualstudio.com/items?itemName=Lencerf.beancount), with syntax checking (bean-check) and support for accounts, currencies, etc. It not only allows selecting existing open accounts but also displays their balance and other metadata. Quite helpful!

    -   [<u>Beancount Formatter</u>](https://marketplace.visualstudio.com/items?itemName=dongfg.vscode-beancount-formatter), which can format the whole document, aligning the numbers, etc. using bean-format.

## If You Have Problems<a id="if-you-have-problems"></a>

If you run into any installation problems, [<u>file a ticket</u>](https://github.com/beancount/beancount/issues/), [<u>email me</u>](mailto:blais@furius.ca), or hit the [<u>mailing-list</u>](https://groups.google.com/forum/#!forum/beancount).

## Post-Installation Usage<a id="post-installation-usage"></a>

Normally the [<u>scripts located under beancount/bin</u>](https://github.com/beancount/beancount/tree/v2/bin) should be automatically installed somewhere in your executable path. For example, you should be able to just run “bean-check” on the command-line.

## Appendix<a id="appendix"></a>

**If everything works, you can stop reading here.** Here I just discuss the various dependencies and why you need them (or why you don’t, some of them are optional). This is of interest to developers and some of this info might help troubleshoot problems if you encounter any.

### Notes on Dependencies<a id="notes-on-dependencies"></a>

#### Python 3.5 or greater<a id="python-3.5-or-greater"></a>

Python 3.5 is widely available at this point, released more than a year ago. However, my experience with open source distribution tells me that a lot of users are running on old machines that won’t be upgraded for a while. So for those users, you might have to install your own Python… don’t worry, installing Python manually is pretty straightforward.

On Mac, it can also be installed with one of the various package management suits, like Brew, e.g., with “`brew install python3`”.

On an old Linux, download the source from [<u>python.org</u>](https://www.python.org/downloads/), and then build it like this:

    tar zxcf Python-3.5.2.tgz
    cd Python-3.5.2
    ./configure
    make
    sudo make install

This should just work. I recommend you install the latest 3.x release (3.5.2 at the time of this writing).

Note: The reason I require at least version 3.5 is because the cdecimal library which supplies Beancount with its implementation of a Decimal representation has been added to the standard library. We need this library to represent all our numbers. Also, Beancount uses the “typing” type annotations which are included in 3.5 (note: You may be able to install “typing” explicitly if you use an older version; no guarantees however).

Some users have reported that there are distributions which package the cdecimal support for Python3 separately. This is the case for Arch, and I’ve witnessed missing cdecimal support in some Ubuntu installs as well. A check has been inserted into Beancount in December 2015 for this, and a warning should be issued if your Python installation does not have fast decimal numbers. If you are in such a situation, you can try to install cdecimal explicitly, like this:

    sudo pip3 install m3-cdecimal

and Beancount will then use it.

#### Python Libraries<a id="python-libraries"></a>

If you need to install Python libraries, there are a few different ways. First, there is the easy way: there is a package management tool called “pip3” (or just “pip” for version 2 of Python) and you install libraries like this:

    sudo pip3 install <library-name>

Installing libraries from their source code is also pretty easy: download and unzip the source code, and then run this command from its root directory:

    sudo python3 setup.py install

Just make sure the “python3” executable you run when you do that is the same one you will use to run Beancount.

Here are the libraries Beancount depends on and a short discussion of why.

##### python-dateutil<a id="python-dateutil"></a>

This library provides Beancount with the ability to parse dates in various formats, it is very convenient for users to have flexible options on the command-line and in the SQL shell.

##### bottle<a id="bottle"></a>

The Beancount web interface (invoked via `bean-web`) runs a little self-contained web server accessible locally on your machine. This is implemented using a tiny library that makes it easy to implement such web applications: [<u>bottle.py</u>](http://bottlepy.org/).

##### ply<a id="ply"></a>

The query client (`bean-query`) which is used to extract data tables from a ledger file depends on a parser generator: I use Dave Beazley’s popular [<u>PLY</u>](http://www.dabeaz.com/ply/) library (version 3.4 or above) because it makes it really easy to implement a custom SQL language parser.

##### lxml<a id="lxml"></a>

A tool is provided to bake a static HTML version of the web interface to a zip file (`bean-bake`). This is convenient to share files with an accountant who may not have your software installed. The web scraping code that is used to do that used the lxml HTML parsing library.

#### Python Libraries for Export (optional)<a id="python-libraries-for-export-optional"></a>

##### google-api-python-client<a id="google-api-python-client"></a>

Some of the scripts I use to export outputs to Google Drive (as well as scripts to maintain and download documentation from it) are checked into the codebase. You don’t have to install these libraries if you’re not exporting to Google Drive; everything else should work fine without them. These are thus optional.

If you do install this library, it require recent (as of June 2015) installs of

-   [<u>google-api-python-client</u>](https://github.com/google/google-api-python-client): A Python client library for Google's discovery based APIs.

-   [<u>oauth2client</u>](https://github.com/google/oauth2client): This is a client library for accessing resources protected by OAuth 2.0, used by the Google API Python client library.

-   [<u>httplib2</u>](https://github.com/jcgregorio/httplib2): A comprehensive HTTP client library. This is used by oauth2client.

These are best installed using the pip3 tool.

Update: as of 2016, you should be able to install all of the above like this:

    pip3 install google-api-python-client

**IMPORTANT:** The support for Python3 is fairly recent, and if you have an old install you might experience some failures, e.g. with missing dependencies, such as a missing “gflags” import. Please install those from recent releases and you should be fine. You can install them manually.

#### Python Libraries for Imports (Optional)<a id="python-libraries-for-imports-optional"></a>

Support for importing identifying, extracting and filing transactions from externally downloaded files is built into Beancount. (This used to be in the LedgerHub project, which has been deprecated.) If you don’t take advantage of those importing tools and libraries, you don’t need these imports.

##### python-magic (optional)<a id="python-magic-optional"></a>

Beancount attempt to identify the types of files using the stdlib “mimetypes” module and some local heuristics for types which aren’t supported, but in addition, it is also useful to install libmagic, which it will use if it is not present:

    pip3 install python-magic

Note that there exists another, older library which provides libmagic support called “filemagic”. You need “python-magic” and not “filemagic.” More confusingly, under Debian the “python-magic” library is called “filemagic.”

##### Other Tools<a id="other-tools"></a>

It is expected that the user will build their own importers. However, Beancount will provide some modules to help you invoke various external tools. The dependencies you need for these depends on which tool you’ve configured your importer to use.

#### Virtualenv Installation<a id="virtualenv-installation"></a>

If you’d like to use virtualenv, you can try this (suggestion by Remy X). First install Python 3.5 or beyond[^2]:

    sudo add-apt-repository ppa:fkrull/deadsnakes
    sudo apt-get update
    sudo apt-get install python3.5
    sudo apt-get install python3.5-dev
    sudo apt-get install libncurses5-dev

Then install and activate virtualenv:

    sudo apt-get install virtualenv
    virtualenv -p /usr/bin/python3.5 bean
    source bean/bin/activate
    pip install package-name
    sudo apt-get install libz-dev  # For lxml to build
    pip install lxml
    pip install beancount

### Development Setup<a id="development-setup"></a>

#### Installation for Development<a id="installation-for-development"></a>

For development, you will want to avoid installing Beancount in your Python distribution and instead just modify your `PYTHONPATH` environment variable to run it from source:

    export PYTHONPATH=/path/to/install/of/beancount

Beancount has a few compiled C extension modules. This is just portable C code and should work everywhere. For development, you want to compile the C modules in-place, within the source code, and load them from there. Build the C extension module in-place like this:

    python3 setup.py build_ext -i

Or equivalently, the root directory has a Makefile that does the same thing and that will rebuild the C code for the lexer and parser if needed (you need flex and bison installed for this):

    make build

With this, you should be able to run the executables under the `bin/` subdirectory. You may want to add this to your `PATH` as well.

#### Dependencies for Development<a id="dependencies-for-development"></a>

Beancount needs a few more tools for development. If you’re reading this, you’re a developer, so I’ll assume you can figure out how to install packages, skip the detail, and just list what you might need:

-   [**<u>nosetests</u>**](https://pypi.python.org/pypi/nose/) (**nose**, or **python3-nose**): This is the test driver to use for running all the unit tests. You definitely need this.

-   [**<u>GNU flex</u>**](https://www.gnu.org/software/flex/): This lexer generator is needed if you intend to modify the lexer. It generated C code that chops up the input files into tokens for the parser to consume.

-   [**<u>GNU bison</u>**](http://www.gnu.org/software/bison/): This old-school parser generator is needed if you intend to modify the grammar. It generates C code that parses the tokens generated by flex. (I like using old tools like this because they are pretty low on dependencies, just C code. It should work everywhere.)

-   [**<u>GNU tar</u>**](http://www.gnu.org/software/tar/): You need this to test the archival capabilities of bean-bake.

-   [**<u>InfoZIP zip</u>**](http://www.info-zip.org/) (comes with Ubuntu): You need this to test the archival capabilities of bean-bake.

-   [**<u>lxml</u>**](http://lxml.de): This XML parsing library is used in the web tests. (Unfortunately, the built-in one fails to parse large XML files.)

-   [**<u>pylint</u>**](http://www.pylint.org/) &gt;= 1.2: I’m running all the source code through this linter. If you contribute code with the intent of it being integrated and released, it has to pass the linter tests (or I’ll have to make it pass myself).

-   [**<u>pyflakes</u>**](https://github.com/pyflakes/pyflakes): I’m running all the source code through this logical error detector. If you contribute code with the intent of it being integrated and released, it has to pass those tests (or I’ll have to make it pass myself).

-   [**<u>snakefood</u>**](http://furius.ca/snakefood/): I use snakefood to analyze the dependencies between the modules and enforce some ordering, e.g. core cannot import from plugins, for example. There’s a target to run it from the root Makefile.

-   [**<u>graphviz</u>**](http://www.graphviz.org/): The dependency tree of all the modules is generated by snakefood but graphed by the graphviz tool. You need to install it if you want to look at dependencies.

I think that’s about it. You certainly don’t need *everything* above, but that’s the list of tools I use. If you find anything missing, please leave a comment, I may have missed something.

[^1]: Some people have [<u>reported</u>](https://www.google.com/url?q=https://groups.google.com/d/msgid/beancount/51fec791-1c38-46e8-b152-08c49e7686c3%2540googlegroups.com?utm_medium%3Demail%26utm_source%3Dfooter&sa=D&ust=1461188801128000&usg=AFQjCNH8StkxqPYKLKMhir6YAz6rXCOKsQ) bugs with the cdecimal library in 3.4. I would recommend actually installing 3.5, which appears to have fixed the problem. Technically, 3.3 will still run, but I’ll deprecate it for 3.5 at some point, probably when the Ubuntu and Mac OS X distributions have it installed by default.

[^2]: Installing py3.5: <http://www.jianshu.com/p/4f4b2ed568f4>
