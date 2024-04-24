Sony A7S all-sky camera
==================================

+------------------------+----------------------------------------------------+
| Where is the data?     | Quicklook images at ``birkeland``, archive at NIRD |
+------------------------+----------------------------------------------------+
| Who?                   | Dag (PI) and Mikko                                 |
+------------------------+----------------------------------------------------+
| Where?                 | Module 3                                           +
+------------------------+----------------------------------------------------+
| Main operational task  | Starting and stopping of the imaging               |
+------------------------+----------------------------------------------------+

.. warning:: The first Sony images up to and including 14 Nov 2015 are out of focus

.. warning:: The date and time in the EXIF-data in each JPEG-image are **incorrect**. The correct date and time
  are embedded in the filename.

Data description
################

The Sony camera stores colour all-sky images in JPEG-format. The image cadence is roughly one image every 10-12 seconds. 
Also, there are quicklook images for realtime webpage: these have a small label indicating date and time of the image. 

The summary plots are keograms covering the dark time of each day.

.. note:: The filenaming convention is a bit confusing, but the raw files are ``LYR-Sony-ddmmyy-HHMMSS.jpg`` where ``dd`` and ``mm`` are
  the day and month, ``yy`` are the last two digits of the year and ``HHMMSS`` are the hours, minutes and seconds of the UT time 
  (start of exposure). The keograms are named slightly differently with a full four digit year ``LYR-Sony-yyyymmdd.png``.

The data from Sony is archived to `NIRD <https://www.sigma2.no/data-storage>`_.


Star calibration
################

Examples and source code for doing a simple star calibration can be found 
at `github <https://github.com/mikkosyrjasuo/UNIS-starcalibration>`_

------------

Computer setup
##############

Control computer
   * Ubuntu 14.04
   * Use ssh for remote access

.. note:: When starting the imaging software, wait for a password prompt! This may take a while...
.. note:: You need to use the imaging software GUI, which requires all sorts of ssh-tunneling via VNC etc. (which does not seem to work well from Windows...)


-------

Configuration
##############

.. note:: The Sony camera body does not include a GPS and the current imaging software does not check nor update
          the date and time in the camera. This results in incorrect EXIF-data in the images. One should set the
          date and time to match the season when powering up the camera at the start of the season.

The configuration of the python program is done in the ``.pysces_asi/settings.txt`` file.

As of December, 2020, the following settings for exposure time etc. are in use:

+-----------+-------------------------+-------+---------------+--------------------+
| Mode      | Min time between images | ISO   | Shutter speed | Notes              |
+===========+=========================+=======+===============+====================+
| Nighttime | 10s                     | 16000 | 4 s           |                    |
+-----------+-------------------------+-------+---------------+--------------------+
| Day Time1 | 30s                     | 100   | 1/1000s       | Not actually used? |
+-----------+-------------------------+-------+---------------+--------------------+
| Day Time2 | 30s                     | 100   | 1/2000s       | Not actually used? |
+-----------+-------------------------+-------+---------------+--------------------+
| Twilight1 | 30s                     | 100   | 1/8s          |                    |
+-----------+-------------------------+-------+---------------+--------------------+
| Twilight2 | 30s                     | 200   | 1s            |                    |
+-----------+-------------------------+-------+---------------+--------------------+
| Twilight3 | 30s                     | 2000  | 1.3s          |                    |
+-----------+-------------------------+-------+---------------+--------------------+

The mode is selected based on the location of the Sun and the Moon in the sky, but not on the phase
of the Moon. One should also note that in LYR the Moon is always less than 40 degrees above the horizon
during wintertime... Here is the selection logic::

   SUN_ANGLE < 0 and SUN_ANGLE > -3 and MOON_ANGLE < 40 = "Twilight1"
   SUN_ANGLE < -3 and SUN_ANGLE > -6 and MOON_ANGLE < 40 = "Twilight2"
   SUN_ANGLE < -6 and SUN_ANGLE > -9 and MOON_ANGLE < 40 = "Twilight3"
   SUN_ANGLE < -9 and MOON_ANGLE < 40 = "Night Time"

There is a backup system disk image in KHO-WD4

Software
########

The ``paskil`` software was originally developed by Nial Peters (2009). For details, see his Master's thesis 
`<http://urn.nb.no/URN:NBN:no-22683>`_

There are also ``pysces_asi`` and ``paskil`` python libraries being used. There are copies in NIRD and 
in github (`<https://github.com/paalge>`_)


   * Instructions for starting and stopping of the camera operations are as the desktop background image. 
     Originals can be found in ``khodata:\Doc\Instrument instruction sheets\Sony_a7s``
   * The software will ask for password for various network drives. It may take some time before the 
     dialog window appears (i.e. several minutes)
   * Data management routines are operated via ``crontab``

------

Drawbacks in the current implementation
#######################################

The current software and setup is a result of incremental additions to the original software. Especially the data
management routines are a bit convoluted and reflect various changes in KHO infrastructure (local disk, NAS-box, NIRD).
Also, ``pysces_asi`` writes huge log files, which is probably a left-over from earlier debugging efforts.

The scheduler appears to be free-running rather than tighly synchronised to system time. This makes the timestamps in filenames inconsistent, which then requires additional care when working with data. The directory structure for data and how the routines pack images into tar-files for archival is somewhat peculiar.

For example, here is what comes out when unpacking archived tar-files ``20200107.tar.gz`` (note missing directories for year and month)::

   06/
   06/Images/
   06/Images/LYR-Sony-070120_133004.jpg
   06/Images/LYR-Sony-070120_133015.jpg
   06/Images/LYR-Sony-070120_133027.jpg
   06/Images/LYR-Sony-070120_133038.jpg
   06/Images/LYR-Sony-070120_133050.jpg
   06/Images/LYR-Sony-070120_133102.jpg
   06/Images/LYR-Sony-070120_133113.jpg
   06/Images/LYR-Sony-070120_133125.jpg
   06/Images/LYR-Sony-070120_133137.jpg
   06/Images/LYR-Sony-070120_133148.jpg
   06/Images/LYR-Sony-070120_133200.jpg

One more practical issue is that the date format in the filename is day-month-(incomplete)-year . Perhaps, in future we might have time to
re-write parts of the code to result in more user-friendly archived data packages. Also, the image capture times could
be consistently fixed to, for example, every 10 seconds during nighttime::

   2020/01/LYR-Sony-20200107_133000.jpg
   2020/01/LYR-Sony-20200107_133010.jpg
   2020/01/LYR-Sony-20200107_133020.jpg
   2020/01/LYR-Sony-20200107_133030.jpg
   2020/01/LYR-Sony-20200107_133040.jpg
   2020/01/LYR-Sony-20200107_133050.jpg
   2020/01/LYR-Sony-20200107_133100.jpg

