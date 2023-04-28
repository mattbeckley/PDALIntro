# Necessary software
- [PDAL](https://pdal.io/en/2.5.3/about.html). The workshop will follow the markdown tutorials using a command line version of PDAL.  Users are expected to have a working version of PDAL so that they can copy and paste commands from the tutorial into their PDAL installation. It is recommended that users use conda to install PDAL.  Details on how to do this are below

- [GDAL](https://gdal.org/download.html#conda). Portions of  the tutorial will utilize GDAL to grid point clouds, look at metadata, perform raster math, etc. It is recommended that users use conda to install GDAL.  Details on how to do this are below.

- [git](https://github.com/git-guides/install-git). Cloning of the git repo is recommended so that users have local access to the data, pipelines, and markdown documents for the workshop.  If it is difficult or not possible to install git and clone the repo, users can always download a zip file directly from github repo so that workshop materials are available locally:

![Github Download Zip](./images/GitDownloadZip.png)



# Conda set up
Conda is an open source package management system and environment management system that runs on Windows, macOS and Linux. Conda quickly installs, runs, and updates packages and their dependencies. [Miniconda](https://docs.conda.io/en/latest/miniconda.html) is a free, light-weight version of conda.  It is recommended to use miniconda to install PDAL and other packages for this workshop.  

Download the conda installer for your OS setup. https://docs.conda.io/en/latest/miniconda.html

After installing conda, create an isolated workspace for this workshop.  
`conda create --name pdalworkshop`
`conda activate pdalworkshop`

Install PDAL, GDAL, jq, and jupyter notebook via conda:

`conda install -c conda-forge pdal jq gdal notebook nb_conda_kernels `

# Jupyter notebook set up
A bash notebook with PDAL commands will be available. Users who wish to use this notebook will need to install the bash_kernel. To enable a bash kernel in a jupyter notebook environment:

`conda activate pdalworkshop`

`pip install bash_kernel ; python -m bash_kernel.install`


# Optional

- [jq](https://stedolan.github.io/jq/): a useful JSON parser.  Best installed with conda (see command above)
- jupyter notebook. Best installed with conda (see command above)
- GIS.  It will be useful to have a GIS (e.g. QGIS, ArcGIS, Global Mapper) to visualize some of the raster outputs.  Users who do not have a GIS can download an install the freely available QGIS software: https://www.qgis.org/en/site/forusers/download.html

