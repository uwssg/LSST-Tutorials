# PHOSIM quick start
[Here](https://confluence.lsstcorp.org/display/PHOSIM/Using+PhoSim) is the most comprehensive phosim documentation I know of.

## Get phosim
This is hosted on bitbucket: [here are tarballs](https://bitbucket.org/phosim/phosim_release/downloads).  You can, of course, clone it and checkout a tag.  This is what I typically do.

## Build phosim
I usually build against a DM stack.  The stack provides cfitsio and fftw (and Eigen, although this doesn't seem to actually be needed).

 * Setup your stack (I'm assuming you also have sims installed)
 * Find the relevant paths: e.g. `$> echo $CFITSIO_DIR`
 * Go into your phosim distribution.
 * There is a configure script in phosim
  * `$> ./configure`
  * This will ask for one of three options.  I use option 'c'.  That's the one that lets you specify the locations to the libraries.
  * Answer the questions.  If it's asking for a library, it will be /path/to/library/lib and if it's asking about headers, it will be /path/to/library/include.
 * You should then be able to say `$> make`

## Use phosim
You can use phosim right out of the box.  E.g. `./phosim examples/star`.  This is because phosim comes with a few example SEDs.

In order to really use phosim with CatSim, you'll want to have the SEDs CatSim refers to.  SEDs from CatSim come in the sims_sed_library package.
 * Go to the phosim data directory: `$> cd data/SEDs`
 * Create links to the distributed SED library.  In Bash, you can do something like: ```$> for i in `ls -d $SIMS_SED_LIBRARY_DIR/*SED`; do  ln -s $i; done```

Now you are in a position to use the CatSim package to make catalogs for you to run through phosim.

## Survival guide
### By default *everything* is turned on.
This means brighter/fatter, tree rings, annealing errors, clouds, variable water vapor, etc.  This means that you will almost always want a 'physics command override' (PCO) file.  This file can do a lot of things and is the main configuration interface to the runtime configuration.  There are even some I/O debuging options you can turn on with this file.  Following is an example minimal PCO file that doesn nothing more than turn on the option to output a debugging file that will report the total number of photons simulated for each object and the centroid of each object (this is per chip).
```
centroidfile 1
```

### Simulating the background is very slow.
Because of the way phosim is architected, some per visit parameters are not set in the instance file.  This means that by default, phosim will choose a dark sky background for you.  Because it just chooses from a Gaussian distribution with sigma=0.9, the sky brightness can be surprisingly large.  You can set this by adding the zenith_v parmeter.  This is the V-band sky brightness at zenith in mag/arcsec**2.  This can be used to turn off the background.  A file with a reasonable background would look like the following:
```
centroidfile 1
zenith_v 22.8
```

### Clouds are everywhere
Cloud opacity (not reflection) is turned on by default.  Like the sky brightness, this is not something you can set from the instance catalog.  You can modify each of the cloud layers independently, but more frequently, I turn them off completely.  Phosim has some convenience commands that help turn off everything associated with an effect.
```
centroidfile 1
zenith_v 22.8
clearclouds
```

### Check your spelling
Phosim generally ignores unknown commands.  This is true in both the instance and PCO files.  The result is that if you misspell one of the commands, it will be silently ignored and you will only have a chance to notice after it has run through.

### Scaling up
The `-s` command line argument allows you to specify the chip to simulate in the format `R??_S??`.  This is useful to split jobs among cores, and also to do a test run.  I also find it useful in situations where my input catalog may have incomplete coverage on some chips.

Phosim uses lots of intermediate files.  These can collide if you are running multiple instances on the same physical system.  There is a `-w` argument that should allow multiple jobs to run in parallel on the same installation, but I have not played with it much.  Instead, I copy the installation and run one instance per copy.

### Eimages
The so-called e-images are output by phosim by default.  They are a really nice starting place if you don't want to go through the hastle of simulating calibration products (darks, flats).  They are essentially trivially ISR'd images.  They have all the astrophysical, atmospheric, and optical effects.  Some electronic effects are in them: bfe and treerings, but not all: e.g. no flatfielding is necessary (I don't think).
