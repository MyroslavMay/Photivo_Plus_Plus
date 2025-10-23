# Photivo Image Processor

Photivo is a multi platform photo processor for raw and bitmap images with
16 bit precision.

Photivo tries to give the user as much control as possible to express their
creativity and to allow flexible adjustments for the various needs in
photography.


## Build and Install

For a detailed build guide see the [wiki](https://web.archive.org/web/20241102114645/https://photivo.org/download).
Pick the appropriate subpage for your operating system.

Photivo uses CMake as its build systems. Build as usual:

    cd build-dir
    cmake /path/to/photivo-repo -DCMAKE_BUILD_TYPE=Release
    cmake --build .
    cmake --install .

These Photivo specific CMake options (`-D`) are available:

* `WITH_SYSTEM_CIMG` (boolean, default: OFF)<br>
  Enables either the system provided CImg library or the one bundled with Photivo.

* `WITH_CLEAR` (boolean, default: ON)<br>
  Builds the *ptClear* program for clearing Photivo user configuration.

* `WITH_ADOBE_PROFILES` (boolean, default: OFF)<br>
   Builds the *Adobe profiles creator* program. Only needed for Photivo developers.

* `WITH_GIMP_TO_PT` (boolean, default: ON)<br>
  Installs the GIMP-to-Photivo GIMP plugin. Needed for transferring images
  from GIMP directly to Photivo.

* `WITH_OLD_PT_TO_GIMP` (boolean, default: OFF)<br>
  Builds the old Photivo-to-GIMP GIMP plugin. Only needed for GIMP versions 2.8 and older,
  which could not load images transferred from Photivo including all their metadata.
  GIMP 2.10 does not have this limitation any more and does not need the plugin.

* `PT_QT_MAJOR_VERSION` (integer between 5 and 9, default: 6)<br>
  Defines which major version of Qt Photivo is built against.

Also an `uninstall` target is provided to undo an earlier `install`. However, uninstalling only
removes files, not empty directories.


## Licence

Photivo is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License version 3
as published by the Free Software Foundation.

!!! Note version 3 only! Not: “or any later version” !!!

Photivo is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with Photivo (see file COPYING).  If not, see 
<http://www.gnu.org/licenses/>.
