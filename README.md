# Configuration for imaging plants with RaspiCams using `raspistill`, `cron`, and `rsync`

These files were written
by J. Steen Hoyer, Carrington lab.
He used similar files
for top-down timelapse imaging
of the vegetative growth
of hundreds of *Arabidopsis thaliana* plants
in several multiweek experiments.

## Purpose

The scripts and cron tables here
are presented as an example
of a minimal imaging configuration
and as research documentation.
It is very unlikely
that you will be able to use them directly
without substantial modification.
In particular,
the hostnames for the twelve raspis
(e.g. `ch129-pos01`)
are hardcoded into the files.
We suggest
reading the cron table files
and shell scripts.
If they seem useful,
modify them to serve your purpose
(adjust hardcoded hostnames etc.)
and then use them to initiate photo capture
with some moderate number of Pi/Camera rigs
and (optionally) automate transfer of the files
to a remove server.
These files support a manuscript in preparation
by Hoyer et al.

We use standard GNU/Linux utilities
to keep things simple
and easy to modify,
in keeping with the Raspberry Pi spirit.
We encourage plant biologists
to experiment with `cron` and `rsync`&#x2014;they are useful
in a wide variety of situations!
RaspiCam imaging
is a great setting
for developing and practicing command line skills.

Copyright © 2017
[Donald Danforth Plant Science Center](https://www.danforthcenter.org/).
See <LICENSE-MIT>.

### Links

Useful pages
under [RaspberryPi.org/documentation/](https://www.raspberrypi.org/documentation/):
-   [configuration/camera.md](https://www.raspberrypi.org/documentation/configuration/camera.md)
-   [usage/camera/raspicam/timelapse.md](https://www.raspberrypi.org/documentation/usage/camera/raspicam/timelapse.md)
-   [hardware/camera/README.md](https://www.raspberrypi.org/documentation/hardware/camera/README.md)
-   [raspbian/applications/camera.md](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md)

We recommend examining
examples of similar approaches:
-   <http://phenotiki.com> &#x2013; includes tools for configuration via a web browser.
-   Ansible-based configuration for a larger-scale experiment:
    1.  <https://github.com/calizarr/EPSCoR_Bramble_GH9C/>
    2.  Simplified version for topdown RasPiCam imaging:
        <https://github.com/maliagehan/gehan-bramble>

These files here are used for acquiring data.
For processing and analysis
of the resulting image files
you may want to use
[PlantCV](http://plantcv.danforthcenter.org/pages/about.html)
or one of the many other available software packages
listed at
[plant-image-analysis.org](http://www.plant-image-analysis.org/)

See [TAIR page](http://www.arabidopsis.org/portals/education/aboutarabidopsis.jsp)
for some information on *A. thaliana*,
including a timelapse video of rosette growth.

## `cron` tables

### Image capture schedule

We install a cron table on each raspi.
There are two variants of this cron table:
<photograph-every-5min.crontab> and
<photograph-with-flips-every-5min.crontab>.
These are nearly identical
but the second one does horizontal and vertical flips
(`-vf -hf` flags)
because five of our Pi/Camera rigs
are oriented 180° opposite the others.
Files get names like '2017-02-10<sub>1630</sub><sub>ch129</sub>-pos01.jpg',
where '1630' indicates 4:30 PM (the last capture of the day)
and 'ch129-pos01' is the hostname
(chamber 129 position #1, above field-of-view #1).
We have imaged plants under short-day conditions
(8 hours of light, from 8:30 AM to 4:30 PM),
with capture throughout the light phase.
See also notes below.

### Lights and wifi

Both cron table variants
turn off wireless connectivity (`ifdown wlan0`)
for most of the night,
to avoid exposing plants to blue LED light
from the USB WiPi dongles we use.
(This will not be an issue if you use
 the built-in wifi module on a model 3 Raspberry Pi board.)
We additionally turn off the green and red indicator LEDs
on each raspi board
by editing the /boot/config.txt file on each raspi:

    dtparam=act_led_trigger=none
    dtparam=act_led_activelow=off
    
    dtparam=pwr_led_trigger=none
    dtparam=pwr_led_activelow=off

With these parameters set,
light from the red LED
effectively becomes an indicator
that a raspi has crashed
(but is still drawing sufficient power to function),
as opposed to simply suffering
from network connectivity problems.

We similarly use a pair of minimal cron tables
to similarly avoid exposing plants to blue light
in between imaging experiments
([1](minimal-light-checks-and-wifi.crontab),
 [2](minimal-light-checks-with-flips-and-wifi.crontab)).

### Image transfer

File transfer is scheduled with a separate cron table
installed on a server:
<pull-images-from-raspis.crontab>.
Syncing photos to a cluster
makes them easier to process/monitor,
and lets us collect more photos
than will fit on a single SD card.

### Helper scripts

Two shell scripts are provided
to streamline starting and ending
image capture on all twelve raspis at once.
-   The [first script](install-twelve-crontabs)
    is used at the start of an experiment
    (or after adjusting a cron table).
    This file copies the appropriate cron table
    onto each of the twelve raspis
    (if necessary)
    and then installs it.
    The script is run without arguments:
    `./install-twelve-crontabs`

-   A second script
    (<install-twelve-mini-crontabs>)
    is used at the end of an experiment (see below)
    to install the minimal cron tables described above
    and thereby suspend image capture.

The next section
describes use of
[a third helper script](take-one-picture-and-pull-it-with-rsync)
for taking and viewing a single snapshot.

## Procedures

### Starting imaging

1.  Take snapshots to convince yourself
    that plants are positioned appropriately.
    This is an excellent time to photograph color standard cards,
    to enable later assessment of sensor drift.
    To take a snapshot with the raspi in position #1,
    run the script like so:
    `./take-one-picture-and-pull-it-with-rsync /path/on/cluster/ 1`
    -   You could run this command (and the next one)
        from your local computer,
        but things will be easier if you run them on a remote server.
2.  Install the correct cron table on each raspi
    (as mentioned above)
    to start regular image capture:
    `./install-twelve-crontabs`
3.  Double check the server cron table.
    Is the correct (hardcoded) destination path on the server specified?
4.  Install the server cron table
    to pull photos:
    `crontab pull-images-from-raspis.crontab`

### Ending imaging

1.  Stop image capture;
    reinstall cron tables that just monitor lights
    and cycle wifi on and off:
    `./install-twelve-mini-crontabs`
2.  If desired, photograph color standard cards,
    as you remove plants
    or shortly thereafter,
    as above:
    `./take-one-picture-and-pull-it-with-rsync /path/on/cluster/ 1`

### Pitfalls

1.  Watch out for color drift
    and consider including standards in your field of view.
    By default, `raspistill` automatically picks
    exposure and color balance settings
    based on a five second video preview.
    This has been sufficient for our purposes
    and provides a starting point
    for testing other settings,
    but it means that the white balance and capture conditions
    can vary over the course of an experiment.
    In particular, the blue rubber mesh
    often placed over soil
    for image-based phenotyping experiments
    (see e.g. [Junker et al. 2014](http://journal.frontiersin.org/article/10.3389/fpls.2014.00770/full#F3),
     Figures 3 and 4)
    can cause color balance "overcompensation",
    resulting in an orange tinge.
    This tinge steadily recedes over the course of an experiment
    (as plant leaves cover the mesh),
    which further complicates image processing.
    -   We embed raw Bayer data
        into JPEG file exif metadata
        (`raspistill -r` flag)
        to enable post-processing,
        but only for the first and last capture of the day.
    -   Lots of room for improvement here!
2.  The clock built into our growth chamber control board
    does not automatically recalibrate itself
    by synchronizing with a server,
    and so the clock steadily drifts forward.
    
    
    Unless the clock is manually corrected,
    the light schedule will eventually shift far enough
    that the first photo of the day will be captured
    before "sunrise".
    
    -   The most reliable way to deal with this issue
        is to manually calibrate the chamber clock
        shortly before the start of every new experiment.
        Adjusting the clock in our growth chamber
        requires shutting it down,
        which in turn necessitates turning off each RasPi.
        (Our twelve RasPis use a GFCI-protected auxiliary power outlet
         built into the growth chamber,
         via an extension cord
         threaded through a port built into the exterior of the chamber.)
        We use [a shell script](shut-down-all-12-raspis)
        to shut down our twelve RasPis,
        and they turn back on automatically after power is restored.
3.  We have used our local timezone in the past,
    but now recommend using Universal Coordinated Time (UTC)
    to avoid potential for confusion and/or loss of data
    caused by the start and end of daylight savings time.
    If you are not using UTC
    (controlled via `raspi-config` internalization settings),
    the start and end of daylight savings time
    may trigger an automatic clock shift on each raspi,
    which can result in the photo capture schedule
    being offset by one hour
    relative to the light cycle.
    -   We have imaged from
        9:30 AM to 5:30 PM local time
        during DST
        (CDT is UTC -0500)
        and 8:30 PM to 4:30 PM
        for the rest of the year
        (CST is UTC -0600),
        These are both equivalent to 1430 to 2230 UTC.
    -   Some growth chamber controllers
        automatically shift the light cycle
        at the start and end of daylight savings time.
        This shift is arguably bad,
        because re-entrainment of plant circadian clocks
        to the new light schedule can alter growth.
        Shifting the start of zeitgeber (ZT) time
        also makes the experiment more difficult to describe.
4.  Make sure your raspis are drawing sufficient power!
    The camera boards draw extra power during photo capture,
    which can cause one or more raspis
    sharing an inadequate power supply to crash.
    See <https://www.raspberrypi.org/documentation/hardware/raspberrypi/power/README.md>

### Monitoring

If desired, one can add an email address
(MAILTO variable)
at the top of the server cron table.
This contact address will then receive an email
every time an rsync transfer fails.
This measure is noisy:
a failed transfer is usually caused
by transient wifi interference,
and merely delays transfer of the relevant files
until the next cycle.
Multiple failed transfers
can indicate that a raspi has crashed,
especially when initial connection was the step that failed.

We additionally use
a [Ganglia dashboard](http://bioinformatics.danforthcenter.org/ganglia/?r=hour&cs=&ce=&c=jcc-pi&h=&tab=m&vn=&hide-hf=false&m=pkts_out&sh=1&z=small&hc=4&host_regex=&max_graphs=0&s=by+name)
for monitoring.
See <http://ganglia.info>

## Plans

Researchers at the Danforth Center
will likely continue using these scripts
for imaging experiments.
We plan to share any improvements we make,
but it is also possible
that we will supplant this code with something else entirely.
To reiterate:
we make these files public
primarily as a learning aid
and as documentation
for related research papers.

Questions, feedback, and contributions
are welcome
via GitHub,
Bitbucket,
or GitLab.com.
