---
title: "HORIZONS Scripting Interface"
description: "How to fetch solar system ephemeris data for free!"
date: "2021-10-26"
file-modified: "2022-08-13"
author: "Joe(y) Carpinelli"
image: "/images/jpl-horizons-scripts.png"
categories:
  - Space
  - Ephemeris
  - Tutorial
  - Technical
aliases:
  - /blog/posts/jpl-horizons-scripts.html
format:
  html: default
---

:::{.callout-note appearance="simple"}

This tutorial describes an [indefinitely supported](https://ssd.jpl.nasa.gov/horizons/manual.html#command-line), 
yet obselete automated command-line interface to the JPL HORIZONS system. The recommended methods for automatically
downloading JPL HORIZONS ephemeris data is the [graphical interface](https://ssd.jpl.nasa.gov/horizons/app.html#/), 
or the new [REST API](https://ssd-api.jpl.nasa.gov/doc/horizons.html).

:::

## Overview

NASA provides fantastic tools that are available for personal use – one such
 tool is NASA JPL's [HORIZONS](https://ssd.jpl.nasa.gov/horizons.cgi). 
HORIZONS allows users to request positions, velocities, and physical attributes for thousands of 
bodies in our solar system. Users can request celestial body positions 
and velocities (often referred to as _ephemeris_) for several time steps. 
These orbital states are very accurately modeled! Ephemeris models 
often include relativistic effects, solar radiation pressure, aspherical 
bodies, and more. After receiving a _grid_ of orbital states
from HORIZONS, a user can _interpolate_ between timesteps 
to find precise orbital states at any time.

HORIZONS has a [web interface](https://ssd.jpl.nasa.gov/horizons.cgi),
and [email and Telnet](https://ssd.jpl.nasa.gov/?horizons)
interfaces. While the web interface is simple to use, it 
does not provide every capability offered by the email and Telnet
interfaces. __If you just want one ephemeris file for a specific use
case, the web interface is probably ideal!__ While I've never 
used the email interface, I _have_ used the Telnet interface --
the Telnet interface can be invoked by entering `telnet://horizons.jpl.nasa.gov:6775`
in a terminal of your choice. Otherwise,
if you want to batch-request many ephemeris files, read on for a
walkthrough for requesting ephemeris files from HORIZONS within 
a terminal, _without_ manually walking through the Telnet interface!

## Procedure
Rather than manually entering your desired ephemeris file attributes through
HORIZONS' web or Telnet interfaces, NASA JPL has provided open-source scripts
for automatically fetching ephemeris data from JPL servers. The steps below 
provide all the information you need to use these scripts, and wrap them with a 
simple `.sh` script for ease of use.

Note that the rest of this walkthrough requires you use a terminal
within a Unix-like system. If you want to use a Windows machine, consider using 
the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install).

If you follow the directions in this walkthrough verbatim, 
you'll fetch a CSV-formatted ephemeris file without
any text labels, with columns (from left to right): 
Julian day, X Position (km), Y Position (km), Z Position (km),
X Velocity (km/s), Y Velocity (km/s), Z Velocity (km/s).

### Install Dependencies

JPL's scripts use `expect`, an automation tool, to interact with the HORIZONS
Telnet interface (so you don't have to). You will need both `expect` and `telnet`
installed on your system. JPL's output ephemeris files contain carriage 
return characters that are not expected by Unix-like systems. 
You can install and use `sponge` (often packaged with `moreutils`), 
as shown below, to help remove these carriage returns, and other
extraneous information from the data files returned by HORIZONS. If you only wish to remove the carriage return characters, you can also simply open the output ephemeris file in [VSCode](https://code.visualstudio.com) -- this will re-format the file for your operating system.

_Installing Dependencies on MacOS:_
```sh
# Uses Homebrew, a MacOs package manager: https://brew.sh
brew install inetutils # provides telnet
brew install expect    
brew install moreutils # (optional) provides sponge
```

_Installing Dependencies on Ubuntu:_

```sh
# Uses apt, the default Ubuntu package manager
sudo apt install telnet
sudo apt install expect
sudo apt install moreutils # (optional) provides sponge
```

Installation on other systems (Debian, Arch, etc.) is likely very similar to the 
installation commands above.

### Download JPL Scripts


JPL HORIZONS maintains scripts which users interact with through the command-line. 
See the HORIZONS [documentation](https://ssd.jpl.nasa.gov/horizons/manual.html#command-line),
which states...

> _The command-line interface can be automated. However, this is no longer recommended 
> given the APIs introduced in 2021. Deprecated example scripts may be found in the 
> [FTP directory](fhttps://ssd.jpl.nasa.gov/ftp/xfr/SCRIPTS/) and will be supported 
> indefinately._

The script interface [`README`](https://ssd.jpl.nasa.gov/ftp/ssd/SCRIPTS/README) 
offers information about the existing scripts. Download and read through this file 
before continuing!

:::{.callout-important appearance="simple"}

You must verify you are permitted to download and use these scripts!
Refer to the HORIZONS [documentation](https://ssd.jpl.nasa.gov/horizons/manual.html#command-line),
the command-line interface [`README`](https://ssd.jpl.nasa.gov/ftp/ssd/SCRIPTS/README),
and the script contents to determine if your usage and context forbids you from using these resources. 
The remainder of this tutorial assumes the user has read these instructions, and has determined 
they meet the qualifications for downloading and using all HORIZONS resources.

:::

If you determine you are permitted, download the `vec_tbl` and `vec_tbl.inp` 
from JPL's servers. The latter sets input 
parameters for your requested ephemeris data, the former is the ultimate script 
users can call to _get_ ephemeris data. Remember to read the HORIZONS documentation 
– usage may not be free for commercial use, and text in each script file may state that 
some users are not permitted to download and use the scripts.

```sh
wget ftp://ssd.jpl.nasa.gov/pub/ssd/SCRIPTS/vec_tbl
wget ftp://ssd.jpl.nasa.gov/pub/ssd/SCRIPTS/vec_tbl.inp
```

We'll also need to make `vec_tbl` executable.

```sh
chmod u+x vec_tbl
```

### Set Input Parameters

The input file, `vec_tbl.inp`, sets environment variables that
`vec_tbl` uses to walk through the HORIZONS Telnet interface 
automatically. These parameters provide information like 
"where should the origin be placed" and "what time window, 
and what size timesteps should be returned".

Edit `vec_tbl.inp` with your desired ephemeris file parameters.
Some helpful parameter values are provided below -- these 
parameters will result in an ephemeris format with the following 
attributes:

* Origin placed at solar system barycenter
* CSV output with Cartesian coordinates
* Time window from 2020 to 2070, with 6-hour increments
* No text labels in the CSV data

Note -- you can replace `@ssb` with `@sun` to 
place the origin of the ephemeris data at the 
Sun's center of mass.

```sh
set   EMAIL_ADDR    "your_email_address@aol.net" ;
set   CENTER        "@ssb"                       ;
set   REF_PLANE     "FRAME"                      ;
set   START_TIME    "2020-Jan-1"                 ;
set   STOP_TIME     "2070-Jan-1"                 ;
set   STEP_SIZE     "6h"                         ;
set   CSV_FORMAT    "YES"                        ;
set   VEC_TABLE     "2"                          ;
set   REF_SYSTEM    "J2000"                      ;
set   VEC_CORR      "1"                          ;
set   OUT_UNITS     "1"                          ;
set   CSV_FORMAT    "YES"                        ;
set   VEC_LABELS    "NO"                         ;
set   VEC_DELTA_T   "NO"                         ;
set   VEC_TABLE     "2"                          ;
```

### Fetch Ephemeris Data

Now we can use `vec_tbl` to download Ephemeris
data for any celestial body tracked by HORIZONS.
The usage for `vec_tbl` is shown below.

```sh
./vec_tbl <BODY_ID> <OUTPUT_FILENAME>
```

The second argument is the NAIF ID for the celestial
body you'd like to track. IDs for common solar system
bodies are provided in the table below.

| Solar System Body      | NAIF ID |
| -----------------      | ------- |
| Mercury Barycenter     | 1       |
| Venus Barycenter       | 2       |
| Earth-Moon Barycenter  | 3       |
| Mars Barycenter        | 4       |
| Jupiter Barycenter     | 5       |
| Saturn Barycenter      | 6       |
| Uranus Barycenter      | 7       |
| Neptune Barycenter     | 8       |
| Pluto Barycenter       | 9       |
| Sun                    | 10      |
| Mercury                | 199     |
| Venus                  | 299     |
| Moon                   | 301     |
| Earth                  | 399     |
| Mars                   | 499     |
| Jupiter                | 599     |
| Saturn                 | 699     |
| Uranus                 | 799     |
| Neptune                | 899     |
| Pluto                  | 999     |

Note the output file will have a preamble, which 
includes physical characteristics of your selected 
celestial body, and other information.

### Re-format Ephemeris Data (optional)

The output format of the Ephemeris file will 
depend on your input parameters provided 
in `vec_tbl.inp`. If you're following 
this walk-through verbatim (aka using
the input parameters provided above),
then your output file will have 7 columns:
Julian day, date-time label, X, Y, Z positions,
X, Y, Z velocities. All columns are numeric, 
with the exception of column 2 -- this makes 
loading data into MATLAB, Python, or Julia
a bit more complicated. We don't need column 2 
to interpolate between Cartesian states,
because column 2 is simply a plain-language description
of the Julian Day value in column 1. We can use 
`sed` and `awk` magic to strip out the second column, 
and remove the preamble from the Ephemeris file. 
The `sed` and `awk` commands below are taken directly
from the two StackOverflow answers: 
[`sed` reference](https://stackoverflow.com/a/38978201),
[`awk` reference](https://unix.stackexchange.com/a/34686).
Remember to replace `<FILE>` with your desired output 
file name! 

```bash
# Filter out data-file preamble
sed -n '/\$\$SOE/,/\$\$EOE/{//!p;}' <FILE> | sponge <FILE> 

# Remove second column (data-time label)
awk -F , 'BEGIN {OFS=FS}  {$2=""; sub(",,", ","); print}' <FILE> | sponge <FILE> 
```

Now your file will have only comma-delimited rows of numbers! 

