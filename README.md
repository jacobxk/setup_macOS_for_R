
# Set Up macOS for R

There's a few blog posts about setting up macOS for R, _e.g._,
["Installing R on OS X – `100% Homebrew Edition'" - Bob Rudis](https://rud.is/b/2015/10/22/installing-r-on-os-x-100-homebrew-edition/);
["Setup OSX for R" - Bhaskar Karambelkar](https://dev.to/bhaskar_vk/setup-osx-for-r)
and ["Install R 100% Homebrew Edition With OpenBlas & OpenMP – My Version" -
Luis Puerto](http://luisspuerto.net/2018/01/install-r-100-homebrew-edition-with-openblas-openmp-my-version/).
This repository is mainly for me to do this quickly when I need to. I've used
a combination of these to get my R installed and running on macOS but as fast
as Homebrew changes the instructions change and there are always gotchas.

I use [Homebrew](https://brew.sh/) to manage my R installation, while it can be
finicky at times, it's been the easiest for me to use to maintain my R
installation and geo packages that I use. _This way of installing R will result
in packages being compiled on installation._ If you do not wish to do this,
there are other methods for installing R using homebrew, _e.g._
`brew cask install r-app`.

This installation will use [OpenBLAS](http://www.openblas.net/), which can lead
to speed-ups and [TinyTeX](https://yihui.name/tinytex/), which
is much smaller to install than the full MacTeX. If you're not comfortable
installing TeX packages, I suggest installing the full MacTex.

## Install XCode Command Line Tools and Homebrew

If you do not have XCode or the XCode command line tools installed, Homebrew will ask you to install it. This is much lighter than installing the full XCode.

Sign in and download the command line tools from Apple.com, https://developer.apple.com/download/more/ and then follow the instructions to install the package.

### Install Header Files

_For Mojave you will also need to install a package containing header files. The package is located at "/Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14. pkg". See, https://forums.developer.apple.com/thread/104296 for more on this._

```bash
sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
```

If you do not have Homebrew installed, you'll need to do this next. Copy/paste
this in the Terminal application. It will download and install Homebrew.

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew analytics off # if you wish not to participate
```

### Install (an updated) Bash and (an updated) nano

macOS comes with Bash and curl, but they're rather outdated. Homebrew can
remedy this for you. iterm2 is just a nicer terminal to work in than the stock
Terminal program supplied by Apple.

```bash
# Install new Bash
brew install bash bash-completion

# Add the new bash to /etc/shells
sudo sh -c 'echo "/usr/local/bin/bash" >> /etc/shells'

# Change your bash shell
chsh -s /usr/local/bin/bash

# Install an updated version of curl
brew install curl

# Get a better terminal
brew cask install iterm2

```

At this point, you can stop, close Terminal and launch iTerm2 and start using it instead.

### Tap Useful Homebrew Taps and Install XQuartz, Java and TinyTeX.

Install XQuartz and Java. XQuartz is required for "full functionality" in R.
The default Homebrew version of R does not include this. Some version of TeX is
required build PDF vignettes in R. TinyTeX, suggested by @robsalasco, is much
smaller and lighter than the full MacTeX and works very nicely with R.

```bash
# Install XQuartz and Java
brew cask install xquartz
brew cask install java

# Export Java settings
echo '
# Setting $JAVA_HOME
export JAVA_HOME="$(/usr/libexec/java_home)"
' >> ~/.bash_profile

# Install TinyTeX, thanks to robsalasco for the suggestion.
curl -sL \
  "https://github.com/yihui/tinytex/raw/master/tools/install-unx.sh" | sh

# This step is necessary to install TeX packages to build R vignettes
tlmgr update --self --all
tlmgr install inconsolata
```

### Install C/C++ Compilers and Libraries

```bash
brew install gcc ccache cmake pkg-config autoconf automake
```

#### Link Compilers

I prefer to use aliases rather than symlinking. This inserts the aliases for
the homebrew version of gcc in your `~/.bash_profile`, which will be used for
installing R and it's packages.

```bash
echo '
# aliases
alias gcc="gcc-8"
alias gcov="gcov-8"
alias g++="g++-8"
alias cpp="cpp-8"
alias c++="c++-8"
' >>  ~/.bash_profile

# Load the new bash shell (this will also see the new gcc aliases loaded)
/usr/local/bin/bash -l
```

### Install libxml

Your R installation will thank you.

```bash
brew install libxml2 libiconv libxslt
brew link libxml2 --force
```

### (Optional) Install Boost

```bash
brew install boost --with-icu4c --without-single
```

### Install OpenBLAS with OpenMP

```bash
brew install openblas --with-openmp
```

### Install R

The basic version of R from Homebrew lacks several capabilities. This tap from
[sethrfore](https://github.com/sethrfore/homebrew-r-srf) fixes that. It's
more like the defunct Homebrew/science tap's version of R. You'll need the
cairo version from this tap R.

```bash
# Tap sethrfore/homebrew-r-srf for R and cairo
brew tap sethrfore/homebrew-r-srf
brew install sethrfore/r-srf/cairo

# Install R
brew install sethrfore/r-srf/r --with-openblas --with-java \
    --with-cairo --with-libtiff --with-pango

# Configure Java for R
R CMD javareconf
```

### Set Up Directory for Package Installations

This will install packages into a local directory and keeps packages as new
versions of R are installed.

```bash
mkdir -p $HOME/Library/R/3.x/library
cat > $HOME/.Renviron <<END
R_LIBS_USER=$HOME/Library/R/3.x/library
END
```

### (Optional) Change Permissions on the R Installed Directory

If you install BioConductor it offers the option to update some of the base
packages. The stock Homebrew installation does not allow the help files
to be written. This changes the permissions to avoid this error.

For most packages, this isn't an issue because the packages are installed
in a local user library. If you aren't installing BioConductor or some
other use case of the sort, it's OK to skip this.

```bash
chmod -R u+w /usr/local/Cellar/r
```

### Install LLVM

LLVM or Low Level Virtual Machine is a library that allow faster compilation of
R packages using OpenMP and also allows those packages to use OpenMP when
we are normally using R.

```bash
brew install llvm
```

### Install Geospatial Libraries

If you use any of R's spatial packages, you'll need these libraries.

```bash
# Install geospatial libraries
brew tap osgeo/osgeo4mac
brew install geos proj
brew install gdal
```

### (Optional) Install SSL/SSH Libraries

```bash
brew install libressl libssh2
```


### Miscellaneous Libraries

These libraries are required by various R packages that I use.

```bash
brew install imagemagick --with-fontconfig --with-ghostscript \
    --with-librsvg --with-pango --with-webp

brew install v8-315 qpdf udunits pandoc pandoc-citeproc jq protobuf libgit2 unixodbc

```

### Set Paths in .bash_profile

This will set up the paths such that when compiling packages in R, R is able to
find the libraries installed by Homebrew.

It also sets the R library up such that you do not need to reinstall every
package when you upgrade R.

```bash
echo '
export PATH="/usr/local/sbin:$PATH"
export PATH="/usr/local/bin:$PATH"
export PATH="/usr/local/opt/gdal2/bin:$PATH"

export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:/usr/local/opt/icu4c/lib/pkgconfig:/opt/X11/lib/pkgconfig"
export GDAL_DRIVER_PATH="/usr/local/lib/gdalplugins"

export R_LIBS_USER="$HOME/Library/R/3.x/library"

# Setting $JAVA_HOME
export JAVA_HOME="$(/usr/libexec/java_home)"
' >>  ~/.bash_profile
```

### Install RStudio and Fira Code

```bash
# Install fira-code font
brew tap caskroom/fonts
brew cask install font-fira-code

# Install RStudio
brew cask install rstudio
```

### Configure R

Set the repository to automatically use RStudio's cloud project as a mirror.

```bash
echo '
local({
  r <- getOption("repos")
  r["CRAN"] <- "https://cran.rstudio.com/"
  options(repos = r)
})
' >> ~/.Rprofile
```

Don't ask me if I want to save the workspace when I exit. Just quit!

```bash
echo '
alias R="R --no-save"
' >> ~/.bash_profile
```

### Install data.table

`data.table` requires
[special instructions](https://github.com/Rdatatable/data.table/wiki/Installation#openmp-enabled-compiler-for-mac)
for macOS when compiling the package as this R installation using Homebrew will
do. Set up the Makevars file to compile data.table:

```bash
mkdir .R

echo '
LLVM_LOC = /usr/local/opt/llvm
CC=$(LLVM_LOC)/bin/clang -fopenmp
CXX=$(LLVM_LOC)/bin/clang++ -fopenmp
CFLAGS=-g -O3 -Wall -pedantic -std=gnu99 -mtune=native -pipe
CXXFLAGS=-g -O3 -Wall -pedantic -std=c++11 -mtune=native -pipe
LDFLAGS=-L/usr/local/opt/gettext/lib -L$(LLVM_LOC)/lib -Wl,-rpath,$(LLVM_LOC)/lib
CPPFLAGS=-I/usr/local/opt/gettext/include -I$(LLVM_LOC)/include
' >> ~/.R/Makevars
```

Install `data.table` just using the terminal. This will launch an R session,
install `data.table` and then exit back to Bash.

```bash
R --vanilla << EOF
install.packages('data.table', repos = 'https://cloud.r-project.org/')
q()
EOF
```

### Set Final Makevars for R

The preceeding Makevars will cause issues when installing some other R packages,
see
[the notes here](https://github.com/Rdatatable/data.table/wiki/Installation#openmp-enabled-compiler-for-mac)
regarding `stringi` in particular.

First delete the Makevars file.

```bash
rm ~/.R/Makevars
```

Then write the new, general-purpose Makevars file.

```bash
echo '
CC=/usr/local/opt/llvm/bin/clang
CXX=/usr/local/opt/llvm/bin/clang++
CFLAGS=-g -O3 -Wall -pedantic -std=gnu99 -mtune=native -pipe
CXXFLAGS=-g -O3 -Wall -pedantic -std=c++11 -mtune=native -pipe
LDFLAGS=-L/usr/local/opt/gettext/lib -L/usr/local/opt/llvm/lib -Wl,-rpath,/usr/local/opt/llvm/lib
CPPFLAGS=-I/usr/local/opt/gettext/include -I/usr/local/opt/llvm/include
' >> ~/.R/Makevars
```

# Updating installed packages

To update the packages in the local library, this command in R will reinstall
all of them after an update to the R installation, _e.g._ 3.4.4 to 3.5.0.

```R
lib <- .libPaths()[1]

pkgs <- as.data.frame(installed.packages(lib),
                      stringsAsFactors = FALSE)$Package

install.packages(pkgs, type = "source")
```

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of
conduct, and the process for submitting pull requests to us.

## Authors

* **Adam H Sparks** - <https://adam.hsparks.github.io/>

## Licence

This project is licensed under the UNLICENSE - see the [LICENSE](LICENSE)
file for details

## Acknowledgments

* boB Rudis, [@hrbrmstr](https://github.com/hrbrmstr)
* Luis Puerto, [@luisspuerto](https://github.com/luisspuerto)
* Bhaskar V. Karambelkar, [@bhaskarvk](https://github.com/bhaskarvk)
* Roberto Salas, [@robsalasco](https://github.com/robsalasco)
