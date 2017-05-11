
![Barakuda Logo](https://brodeau.github.io/barakuda/logo.svg)

Example of a set of webpages generated by Barakuda:
https://brodeau.github.io/barakuda/example/

# Getting-started with Barakuda

## Requirements

* A FORTRAN 90 compiler

* netcdf library with support for the appropriate F90 compiler

* NCO

* 'convert' from ImageMagick if you want to generate GIF movies

* 'ffmpeg' with x264 support if you want to generate MP4 movies

* For time-series and 2D plots, the following up-to-date packages:
  => python-netcdf4 (from netCDF4 import Dataset) and Matplotlib
  => for map projections you'll also need the Basemap package
  
  A good idea is to install a shiny python distribution, something like Canopy:
  => https://www.enthought.com/products/canopy/

  In any case, specify the appropriate "PYTHON_HOME" environment variable in
  your ${BARAKUDA_ROOT}/configs/config_<MYCONF>.sh or ./config_<MYCONF>.sh file

* NEMO output data! => A directory containing the MONTHLY-AVERAGED, global
                       (rebuilt), NEMO output to analyze
  (grid_T, grid_U, grid_V and icemod files) as "*.nc", "*.nc.gz" or ".nc4"

  For NEMO 3.6 and above some appropriate sets of xml configuration files for
  XIOS2 can be found in: src/xios2_xml/

* a NEMO mesh_mask file and the the corresponding basin_mask (ocean basins).
  (variables MM_FILE and BM_FILE into the config_<MYCONF>.sh file you use)
  To create the NEMO mesh_mask.nc just launch the relevant NEMO experiment with the
  namelist parameter nn_msh set to 1 !

  For both ORCA1 and ORCA025 configs (regardless of the number of levels) it is
  safe to use the basin_mask.nc provided in the "data/" sub-directory of Barakuda.
  
  If you want to create your own basin_mask.nc containing your favorite
  seas/regions, proceed as follow:

  1. use the "python/exec/orca_mesh_mask_to_bitmap.py" python script to create a
  black-and-white bitmap image of the land-sea-mask from the mesh_mask.nc
  generated by NEMO.

  2. use your favorite rater image editor (Gimp, PhotoShop, etc) to easily edit
  the bitmap image and create a new image of your sea/region of interest. Save
  it as a "tiff" image!

  3. then you can use "python/exec/tiff_to_orca_mask.py" python script to
  generate the new basin_mask.nc netcdf file out of your tiff images!



## I / Compile CDFTOOLS executables 

 * CDFTOOLS is a set of FORTRAN executables intended to perform a multitude of
   ocean diagnostics based on NEMO output
   (https://github.com/meom-group/CDFTOOLS). However, this is a slightly
   modified light version here...  SO DO NOT USE AN OFFICIAL CDFTOOLS
   DISTRIBUTION, stick to the one that comes with Barakuda!

* move to the 'barakuda/cdftools_light' directory

* configure your own 'make.macro' for your system (some templates for gfortran
  and Intel are provided...)
    => just copy or link your own "macro.your_arch" to "make.macro" !
    => F90 compiler and related netcdf library to use

* compile with 'gmake'

* if that was successful the 'barakuda/bin' directory should contain the 8
  following executables:

        * cdficediags.x
        * cdfmaxmoc.x
        * cdfmhst.x
        * cdfmoc.x
        * cdfpsi.x
        * cdfsigtrp.x
        * cdficeflux.x
        * cdftransportiz.x
        * cdfvT.x

           

## II / Create and configure your own "config_<MY_CONF>.sh"

All setup related to your host, simulation, location of third party files is
defined in the "config_<MY_CONF>.sh" file.

You can either used to chose a config file located in the
"${BARAKUDA_ROOT}/configs" directory of Barakuda:
('${BARAKUDA_ROOT}/configs/config_<MY_CONF>.sh')

Or, in case you have no write access into ${BARAKUDA_ROOT}/ and call the Barakuda
suite of scripts from another location, hereafter "work directory", you can use
a "config_<MY_CONF>.sh" present in the "work directory".

Note: if a given "config_<MY_CONF>.sh" exists both in "${BARAKUDA_ROOT}/configs"
and the "work directory", BaraKuda will always refer to "config_<MY_CONF>.sh"
present in the "work directory".

IMPORTANT: Always refer to the most relevant '${BARAKUDA_ROOT}/configs/config_*_TEMPLATE.sh' file
to design or re-adjust yours! These are symbolic links pointing to the last
officially supported and most up-to-date config files.  It should be
sufficiently well commented for you to be able to adjust your own config file.

MY_CONF should always be of the form: "(e)ORCA<RES>_L<NLEV>_<blabla>.sh"
        ( with NLEV being the number of z levels )

NEMO output files must be monthly averages and of the following form:

        <EXP NAME>_1m_<YEAR>0101_<YEAR>1231_<GRID_TYPE>.nc(.gz)

        (GRID_TYPE=grid_T/grid_U/grid_V/icemod) 

Gzipped or not!

All files for all years must all be saved in the same directory (see
NEMO_OUT_STRCT in the config file). Better if this directory only contains NEMO
output files and nothing else!

Alternatively NEMO files can be saved/organized in sub-directories a la
EC-Earth: (ex: year 1995 of experiment started in 1990 is the 6th year so files for
1995 are saved into sub-directory (of NEMO_OUT_STRCT) "006" (set 'ece_exp' to 1
or 2 then).

If you want to perform the "climatology" plots (see section IV) you will need
monthly "observed" 2D and 3D of T and S (and sea-ice fraction) data interpolated
on the ORCA grid you are using. Usually you should already have them since they
are needed to initialize your simulation (initial state for T & S). These are
the following files in your BaraKuda config file: F_T_OBS_3D_12, F_S_OBS_3D_12,
F_SST_OBS_12, F_ICE_OBS_12.



## III) Create diagnostics


Launch "barakuda.sh"

    ./barakuda.sh -C <MY_CONF> -R <EXP>

    (ex: ./barakuda.sh -C ORCA1_L75_v36_triolith -R SL36C00)

Use the -h switch to see available options



## IV) Create figures and browsable HTML page

### A/ Once the previous job has finished to run, launch

   * To only generate time-series plots use the "-e" switch:

        ./barakuda.sh -C <MY_CONF> -R <EXP> -e

        (ex: ./barakuda.sh -C ORCA1_L75_v36_triolith -R SL36C00 -e)

   * To generate time-series + 2D climatology plots use the "-E" switch,
     provided you have built the monthly/annual climatology (based on N years of
     your simulation) out of your experiment with the "build_clim.sh" script
     (see point V/B):
     
        ./barakuda.sh -C <MY_CONF> -R <EXP> -E

### B/ To be able to create the "climatology" plots (maps, sections, etc, based on a monthly climatology of a few years) you will have to

i) create the climatology with the "build_clim.sh" script:  

    ./build_clim.sh -C <MY_CONF> -R <EXP> -i <first_year> -e <last_year>

(check ./build_clim.sh -h to see the other important options...)

    (ex: ./build_clim.sh -C ORCA1_L75_v36_triolith -R SL36C00 -f 10 -i 0090 -e 0099 -k 4)
      

  ii) the you can tell "barakuda.sh" to create climatology-related plots by
       using the "-E" switch instead of "-e" (see point V/A)


### C/ To compare time-series between at least 2 (already diagnosed) experiments:
   
    ./compare_time-series.sh -C <MY_CONF> -R <EXP1>,<EXP2>,...,<EXPn>

    (ex: ./compare_time-series.sh -C ORCA1_L75_v36_triolith -R SL36C00,SL36EIE )

