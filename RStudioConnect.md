# RStudio Connect: Build Guide

|Property|Value|
|:--|:--|
|OS|Ubuntu 18.04.xx|
|Reference| [RStudio Connect: Admin Guide](https://docs.rstudio.com/connect/admin/index.html)|

## Checklist

- [ ] Install R
- [ ] Install package dependencies for likely R packages 
- [ ] Download & install RStudio Connect
- [ ] Edit RStudio Connect config file
- [ ] Test deployment

## 1. Install R
 Administrators should install the versions of R that they wish to support from source so that user content is run in an environment as close as possible to the development environment. This allows maintenance of multiple versions of R simultaneously and mitigates the risk associated with updating the version of R.
 
To build R from source, first, acquire the build dependencies for R.

In Ubuntu, you can install build dependencies with

```console
sudo apt-get build-dep r-base
```

These may also be helpful later on when you attempt to build R from source

```
sudo apt-get install build-essential
sudo apt-get install libbz2-dev
sudo apt-get install fort77
sudo apt-get install xorg-dev
sudo apt-get install liblzma-dev  libblas-dev gfortran
sudo apt-get install gcc-multilib
sudo apt-get install gobjc++
sudo apt-get install libreadline-dev
```

Second, you should download and unpack the [source tarball](https://cran.r-project.org/src/base/R-3/R-3.6.0.tar.gz) for the version of R that you want to install from CRAN. To install `R-3.6.0`, this might look like the following:

```console
# Download and extract source code
cd /tmp
wget https://cran.r-project.org/src/base/R-3/R-3.6.0.tar.gz
tar -xzvf R-3.6.0.tar.gz
cd R-3.6.0
```

The final step is to configure, make, and install R. The recommended install location for RStudio Connect is at `/opt/R/3.6.0`.

```console
# Build R from source
sudo ./configure \
    --prefix=/opt/R/$(cat VERSION) \
    --enable-memory-profiling \
    --enable-R-shlib \
    --with-blas \
    --with-lapack
	
sudo make
sudo make install
```

**It is very important that the R installation folder is not moved once it is installed from source. Libraries are statically linked, so moving the folder will break the installation of R.**

To test that the installation[^1] went smoothly, execute:

[^1]: If you wish to make this non-standard location for R, the default R install, you need to do two tings

	* First of all, you should make sure that the `R` executable has in fact the required permissions.

		`sudo chmod a+rx /opt/R/3.6.0/bin/R`

	* Then you can create a symbolic link:

		`sudo ln -s /opt/R/3.6.0/bin/R /usr/local/bin`

	Using a link carries some security risks (for example when `/opt/R/3.6.0/bin/R` can be written by non-root users).


```console
/opt/R/3.6.0/bin/R --version
```
Your output should look something like:

```
R version 3.5.3 (2019-03-11) -- "Great Truth"
Copyright (C) 2019 The R Foundation for Statistical Computing
Platform: armv7l-unknown-linux-gnueabihf (32-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under the terms of the
GNU General Public License versions 2 or 3.
For more information about these matters see
http://www.gnu.org/licenses/.

```

-----


## 2. Install package dependencies for likely R packages 

The following system dependencies are required by many common R packages and nearly all deployments will need to provide these. These package names may vary slightly between different versions of Ubuntu.

```
ibcurl4-gnutls-dev
openjdk-7-* # may require also executing `R CMD javareconf`
libxml2-dev
libssl-dev
texlive-full # very large dependency, but needed to render PDF documents from R Markdown
```

Typically, the following will do the trick:

```console
sudo apt-get install libcurl4-gnutls-dev libxml2-dev libssl-dev texlive-full
sudo apt install openjdk-11-jdk
sudo R CMD javareconf
```

-----

## 3. Install RStudio Connect

You will use `gdebi` to install Connect and its dependencies. It is installed via the `gdebi-core` package.

```console
sudo apt-get install gdebi-core
```

You should have a `.deb` installer for RStudio Connect. It can be downloaded from the RStudio website. If you only have a link to this file, you can use `wget` to download the file to the current directory.

```console
wget https://<download-url>/rstudio-connect_1.7.2.2-14_amd64.deb
```

Once the `.deb` file is available locally, run the following command to install RStudio Connect.

```console
sudo gdebi rstudio-connect_1.7.2.2-14_amd64.deb
```

This will install Connect into `/opt/rstudio-connect/`, and create a new rstudio-connect user.

## 4. Edit RStudio Connect config file

RStudio Connect is controlled by the `/etc/rstudio-connect/rstudio-connect.gcfg` configuration file. You will edit this file to make server-wide configuration changes to the system.

Restart RStudio Connect after altering the rstudio-connect.gcfg configuration file.

`sudo systemctl restart rstudio-connect`


