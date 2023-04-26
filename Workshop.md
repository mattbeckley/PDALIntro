# Set up
Use conda to create an isolated workspace for this workshop.  
`conda create --name pdalworkshop`
`conda activate pdalworkshop`

Install PDAL, GDAL, and jupyter notebook via conda:

`conda install -c conda-forge pdal python-pdal jq gdal notebook nb_conda_kernels `

To enable a bash kernel in jupyter:

`pip install bash_kernel ; python -m bash_kernel.install`

If you want use a dockerized version of pdal:

`docker pull pdal/pdal`


# Acknowledgements
The focus of this workshop is getting users comfortable with basic PDAL commands, and workflows. This workshop is geared to the novice PDAL user. Content for this workshop is based on a subset of two existing PDAL workshops:
- PDAL's main workshop material: https://pdal.io/workshop/index.html
- Dr. Adam Steer's PDAL workshop from FOSS4G SotM Oceania in 2018: https://github.com/adamsteer/f4g-oceania-pdal

Users are encouraged to explore these two excellent resources for more advanced PDAL usage and examples.

# Background info on PDAL.  
![PDAL Logo](./images/PDAL_Logo.png)

- PDAL is one of the few open source tools to work with point cloud data
- LasTools is "mostly" free, but advanced usage requires payment.
- PDAL also utilizes and aligns nicely with other open source tools such as GDAL and PROJ.

# Ways of working with PDAL
There are a couple of different pathways to working with PDAL.  The most common are: CLI, PDAL docker containers and PDAL Python API. For most of this workshop we will utilize the command line PDAL.
## CLI
- simplified syntax as compared to dockerized version.
- Can pipe in bash commands when working with large sets of files

```
pdal --version
pdal info --metadata ./data/OR_WizardIsland.laz
```

## Dockerized PDAL
Useful if you want to work avoid OS-specific issues.  Also useful to easily work with different versions of the software: https://hub.docker.com/r/pdal/pdal/tags

```
docker pull pdal/pdal
docker run pdal/pdal pdal --version
docker run -v $PWD/data/:/data pdal/pdal pdal info --metadata /data/OR_WizardIsland.laz
```

Command Breakdown:
- "docker run" - starts the PDAL container.  Need to run everytime a PDAL command is issued 
- "-v" maps the drives. Local drives need to be mapped to a location in the container.  Multiple volumes can be mapped.
- pdal/pdal is the name of the PDAL container.  You can download an image for a specific version, in which case the command would be something like: pdal/pdal:2.5.3
- "pdal info --metadata /data/OR_WizardIsland.laz" is the actual PDAL command.  Note the path to dataset is the path to the mounted volume in the docker container, and NOT on your local machine.

- pros/cons to using Docker.  For simple workflows at the command line, commands can get long and tedious.


## PDAL Python API
- Useful to integrate with existing python code.

```
import pdal
print(pdal.__version__)
```

# PDAL mechanics

## Command line commands vs pipelines.  
- PDAL commands can be issued at the command line with a variety of options

- e.g. Convert a LAZ file to a LAS file in version 1.2:
```
pdal translate --writers.las.minor_version=2 input.laz output_v12.las
```

- e.g. Reproject a file from geographic coordinates (lat/lon) to UTM-based coordinates:
```
pdal translate input_4326.laz output_UTM10N.laz reprojection --filters.reprojection.out_srs="EPSG:32610"
```

- Once commands and workflows get excessively long or complicated, it probably makes sense to start utilizing pipelines.


## Concept of Pipelines.
- PDAL utilizes a JSON input file called a "pipeline" that can execute a series of commands
- Pipelines are useful when building complex workflows.  Each section of the pipeline is called a "stage", and the output of one "stage" will be the input of the following "stage".
- Stages will mostly consist of a reader, writer, or filter operation.

- Here is a simple reprojection operation, but we want to utilize some of the extra optional parameters, so the command would be quite cumbersome to do via a single command line call.  Here we have 3 "stages": a reader, reprojection, and writer stage: 
```
{
    "pipeline": [{"type" : "readers.las",
                  "filename": "input.las"
                 },
                 {"type":"filters.reprojection",
                  "in_srs":"EPSG:26910+5703",
                  "out_srs":"EPSG:6339+5703"
                 },
                 {"type" : "writers.las",
                  "filename": "output.laz",
                  "forward": "header",
                  "compression": "laszip",
                  "a_srs": "EPSG:6339+5703"
                 }
                 ]}
```

- each stage or operation will have its own set of options that you can customize. For example, go to [Writers.las documentation](https://pdal.io/en/2.5.3/stages/writers.las.html) to see what options are available when writing out a LAS/LAZ file.

- In this example, note the use of the "forward" option. Setting this option to different values will help preserve header values from the original input file.  For example, it may be beneficial to save the original creation date for a file.

- To call the pipeline, specify the [pipeline](https://pdal.io/en/2.5.3/pipeline.html#) command to read in the JSON pipeline and executes the commands:

```
>> pdal pipeline pipeline_example.json
```


### Processing Modes
- PDAL has two modes of data processing: standard or stream mode.  In standard mode all input is read into memory before it is processed.  This is often necessary for operations that need access to all of the points at a given time (e.g. sorting).
- Stream mode processes data as it is read in chunks, as is much less memory-intensive.
- When using multi-stage pipelines with large datasets, there is the possibility of out-of-memory issues, especially if some of the operations require standard mode.  In these cases, it may be adviseable to break the pipeline into multiple pipelines.



## Readers, Writers, Filters


# Inspecting a LAS/LAZ file
## Metadata
Use the [info application](https://pdal.io/en/2.4.3/apps/info.html) to get basic info about a file. Using the "--metadata" flag will print out the metadata from the header.

```
>> pdal info --metadata ./data/OR_WizardIsland.laz
{
  "file_size": 27167488,
  "filename": "/Users/beckley/Documents/OT/meetings/RCN_May2023/notebook/data/OR_WizardIsland.laz",
  "metadata":
  {
    "comp_spatialreference": "PROJCS[\"NAD83 / Oregon GIC Lambert (ft)\",GEOGCS[\"NAD83\",DATUM[\"North_American_Datum_1983\",SPHEROID[\"GRS 1980\",6378137,298.257222101,AUTHORITY[\"EPSG\",\"7019\"]],AUTHORITY[\"EPSG\",\"6269\"]],PRIMEM[\"Greenwich\",0,AUTHORITY[\"EPSG\",\"8901\"]],UNIT[\"degree\",0.0174532925199433,AUTHORITY[\"EPSG\",\"9122\"]],AUTHORITY[\"EPSG\",\"4269\"]],PROJECTION[\"Lambert_Conformal_Conic_2SP\"],PARAMETER[\"latitude_of_origin\",41.75],PARAMETER[\"central_meridian\",-120.5],PARAMETER[\"standard_parallel_1\",43],PARAMETER[\"standard_parallel_2\",45.5],PARAMETER[\"false_easting\",1312335.958],PARAMETER[\"false_northing\",0],UNIT[\"foot\",0.3048,AUTHORITY[\"EPSG\",\"9002\"]],AXIS[\"Easting\",EAST],AXIS[\"Northing\",NORTH],AUTHORITY[\"EPSG\",\"2992\"]]",
    "compressed": true,
    "copc": false,
    "count": 5878047,
    "creation_doy": 107,
    "creation_year": 2023,
    "dataformat_id": 1,
    "dataoffset": 1284,
    "filesource_id": 0,
    "global_encoding": 0,
    "global_encoding_base64": "AAA=",
    "gtiff": "Geotiff_Information:\n   Version: 1\n   Key_Revision: 1.0\n   Tagged_Information:\n      End_Of_Tags.\n   Keyed_Information:\n      GTModelTypeGeoKey (Short,1): ModelTypeProjected\n      GTRasterTypeGeoKey (Short,1): RasterPixelIsArea\n      GTCitationGeoKey (Ascii,28): \"NAD83 / Oregon Lambert (ft)\"\n      GeogCitationGeoKey (Ascii,6): \"NAD83\"\n      GeogAngularUnitsGeoKey (Short,1): Angular_Degree\n      ProjectedCSTypeGeoKey (Short,1): Code-2992 (NAD83 / Oregon GIC Lambert (ft))\n      ProjLinearUnitsGeoKey (Short,1): Linear_Foot\n      End_Of_Keys.\n   End_Of_Geotiff.\n",
    "header_size": 227,
    "major_version": 1,
    "maxx": 873448.1,
    "maxy": 438854.95,
    "maxz": 6960.01,
    "minor_version": 2,
    "minx": 870920.41,
    "miny": 436914.9,
    "minz": 6172.44,
    "offset_x": 0,
    "offset_y": 0,
    "offset_z": 0,
    "point_length": 28,
    "project_id": "00000000-0000-0000-0000-000000000000",
    "scale_x": 0.01,
    "scale_y": 0.01,
    "scale_z": 0.01,
    "software_id": "las2las (version 120813) + OT",
    ...
```

- This command is useful to get some basic metadata:
    - Coordinate system of the dataset
    - Point counts
    - min/max of XYZ values
    
- The scale_x, scale_y, and scale_z help determine the precision of the data. LAS stores XYZ values as 32-bit integers, and then applies the scale values to obtain the appropriate precision.  If these parameters are set to an unrealistic value, it could create unnecessarily large files when converting from LAS to LAZ (see article on [LASzip](https://www.cs.unc.edu/~isenburg/lastools/download/laszip.pdf))


## Printing Points 
Use the [info application](https://pdal.io/en/2.4.3/apps/info.html) to get basic info about a file.  Utilizing the -p option lets the user print out a specific point from the file

```
>> pdal info ./data/OR_WizardIsland.laz -p 0

{
  "file_size": 27167488,
  "filename": "./data/OR_WizardIsland.laz",
  "now": "2023-04-19T13:00:12-0600",
  "pdal_version": "2.5.3 (git-version: Release)",
  "points":
  {
    "point":
    {
      "Classification": 2,
      "EdgeOfFlightLine": 0,
      "GpsTime": 500486.8319,
      "Intensity": 49,
      "NumberOfReturns": 1,
      "PointId": 0,
      "PointSourceId": 558,
      "ReturnNumber": 1,
      "ScanAngleRank": 0,
      "ScanDirectionFlag": 1,
      "UserData": 120,
      "X": 870920.64,
      "Y": 438092.59,
      "Z": 6638.35
    }
  },
  "reader": "readers.las"
}

```
## Classifications
Use the [info application](https://pdal.io/en/2.4.3/apps/info.html) with the --stats flag and perform filtering to get a summary of the classifications for a given lidar file. For example:

```
>> pdal info ./OR_WizardIsland.laz --stats --filters.stats.dimensions=Classification 
   --filters.stats.count=Classification
{
  "file_size": 27167488,
  "filename": "OR_WizardIsland.laz",
  "now": "2023-04-25T13:22:28-0600",
  "pdal_version": "2.1.0 (git-version: Release)",
  "reader": "readers.las",
  "stats":
  {
    "statistic":
    [
      {
        "average": 1.621197483,
        "count": 5878047,
        "counts":
        [
          "1.000000/4413629",
          "2.000000/1151988",
          "9.000000/312430"
        ],
        "maximum": 9,
        "minimum": 1,
        "name": "Classification",
        "position": 0,
        "stddev": 1.792156286,
        "variance": 3.211824153
      }
    ]
  }
}
```

- Note the counts section displays the lidar classification and its point count per class.

# Reprojection 
- Use the [reprojection filter](https://pdal.io/en/2.4.3/stages/filters.reprojection.html#filters-reprojection) to reproject datasets:

```
{
    "pipeline": [{"type" : "readers.las",
                  "filename": "input.las"
                 },
                 {"type":"filters.reprojection",
                  "in_srs":"EPSG:26910+5703",
                  "out_srs":"EPSG:6339+5703"
                 },
                 {"type" : "writers.las",
                  "filename": "output.laz",
                  "forward": "header",
                  "compression": "laszip",
                  "a_srs": "EPSG:6339+5703"
                 }
                 ]}
```

- This will reproject the data to a newer NAD83 realization (2011).  Putting the code for the NAVD88 datum in the EPSG code (e.g. 6339+5703) will display the vertical datum info in the metadata.

- older versions of PDAL would be able to apply a geoid by doing the following:

```
{
    "pipeline": [{"type" : "readers.las",
                  "filename": "input_Ellipsoidal.laz"
                 },
                 {"type":"filters.reprojection",
                  "in_srs":"EPSG:32611",
                  "out_srs":"+init=EPSG:32611 +geoidgrids=g2003conus.gtx"
                 },
                 {"type" : "writers.las",
                  "filename": "output_Geoid.laz",
                  "compression": "laszip",
                  "a_srs": "EPSG:32611+5703"
                 }
		]
}
```

- newer versions (>= 2.5) seem to have issues with the PROJ syntax and do not appear to read the geoid grids.

- 

# Creating boundaries of data
- Utilizing the info command, the boundary of a dataset can be obtained by using the "--boundary" flag.  This will output the boundary in WKT format in JSON-formatted output.

```
>> pdal info ./data/Siuslaw.laz --boundary 

{
  "boundary":
  {
    "area": 5083656.235,
    "avg_pt_per_sq_unit": 2.670004261,
    "avg_pt_spacing": 0.9299756138,
    "boundary": "POLYGON ((871585.08 436854.9,871635.22 436898.32,872562.93 436898.32,872613.07 436854.9,872863.8 436854.9,872913.95 436898.32,873465.56 436898.32,873465.56 438852.58,873390.34 438896.0,870920.64 438874.29,870908.1 436898.32,871509.86 436898.32,871585.08 436854.9))",
    "boundary_json": { "type": "Polygon", "coordinates": [ [ [ 871585.07628024998121, 436854.896951399976388 ], [ 871635.222414609976113, 436898.324777669971809 ], [ 872562.925900229951367, 436898.324777669971809 ], [ 872613.07203458994627, 436854.896951399976388 ], [ 872863.802706380025484, 436854.896951399976388 ], [ 872913.948840740020387, 436898.324777669971809 ], [ 873465.556318680057302, 436898.324777669971809 ], [ 873465.556318680057302, 438852.57695966999745 ], [ 873390.33711714996025, 438896.004785929981153 ], [ 870920.64000000001397, 438874.290872799989302 ], [ 870908.103466409957036, 436898.324777669971809 ], [ 871509.857078709988855, 436898.324777669971809 ], [ 871585.07628024998121, 436854.896951399976388 ] ] ] },
    "density": 1.156263667,
    "edge_length": 0,
    "estimated_edge": 43.42782627,
    "hex_offsets": "MULTIPOINT (0 0, -12.5365 21.7139, 0 43.4278, 25.0731 43.4278, 37.6096 21.7139, 25.0731 0)",
    "sample_size": 5000,
    "threshold": 15
  },
  "file_size": 27167488,
  "filename": "./data/OR_WizardIsland.laz",
  "now": "2023-04-25T17:03:10-0600",
  "pdal_version": "2.5.3 (git-version: Release)",
  "reader": "readers.las"
}

```

- However, to get a boundary in vector format to visualize in a GIS or Google Earth requires some additional steps.  The PDAL command, [tindex](https://pdal.io/en/2.5.3/apps/tindex.html#tindex-command) is used to create a boundary that utilizes the [hexbin filter](https://pdal.io/en/2.5.3/stages/filters.hexbin.html#filters-hexbin)

```
>> pdal tindex create --tindex ./data/Siuslaw_bounds.shp --filespec ./data/Siuslaw.laz -f "ESRI Shapefile"
```

- Load the shapefile into a GIS to see its extent:
![Siuslaw Bounds](./images/SiuslawRiverBounds.jpeg)


- For rough estimations of boundaries, this is usually sufficient. To obtain a more precise fit of the data alter some of the parameters in the [filters.hexbin](https://pdal.io/en/2.5.3/stages/filters.hexbin.html#filters-hexbin) command.  The "edge_size" parameter is particularly useful for this scenario as it controls the size of the hexagon boundaries used to estimate whether a section of the dataset should be considered. Finding an appropriate value for edge_size can be an iterative process.  For example, try using a vlue of "50" units (for this dataset, the units are feet).

```
>> pdal tindex create --tindex ./data/Siuslaw_bounds50.shp --filters.hexbin.edge_size=50 --filespec ./data/Siuslaw.laz -f "ESRI Shapefile"

```

- Load the shapefile into a GIS to see its extent:
![Siuslaw Bounds_Fit](./images/SiuslawRiverBounds_Fit.png)

- Note how this is a much better fit to the data, and shows regions where there is no data (over the water). However, use caution with the edge_size parameter as setting it too low might not capture an appropriate amount of data, and mis-represent the data coverage.

# Filtering
- PDAL offers lots of different [filtering operations](https://pdal.io/en/2.5.3/stages/filters.html).

- Removing noise is a basic operation that is often used when working with lidar data.  Standard ASPRS classifications assign low point noise a value of 7, and high point noise a value of 18.  However, sometimes providers don't follow these conventions and may classify noise as a custom value.  For example, the USGS 3DEP data over Tacoma: [WA PierceCounty 1 2020](https://portal.opentopography.org/usgsDataset?dsid=WA_PierceCounty_1_2020) has noisy data with classifications of 135 and 146.  It is often useful to use the Potree 3D point cloud viewer in OpenTopography to visualize the point cloud.

![Fox Island Noise](./images/FoxIslandNoise.png)

- From this visualization, we can see an excessive amount of noise both below and above the ground surface

- We can first do a summary of the classifications, to see what we need to remove:

```
>> pdal info ./data/FoxIsland.laz --stats --filters.stats.dimensions=Classification  --filters.stats.count=Classification
{
  "file_size": 18337307,
  "filename": "FoxIsland.laz",
  "now": "2023-04-26T13:48:53-0600",
  "pdal_version": "2.5.3 (git-version: Release)",
  "reader": "readers.las",
  "stats":
  {
    "statistic":
    [
      {
        "average": 5.633072822,
        "count": 3100062,
        "counts":
        [
          "1.000000/2524244",
          "2.000000/474803",
          "135.000000/69015",
          "146.000000/32000"
        ],
        "maximum": 146,
        "minimum": 1,
        "name": "Classification",
        "position": 0,
        "stddev": 24.4020614,
        "variance": 595.4606004
      }
    ]
  }
}

```

- From this summary, we can see that we probably want to get rid of class 135 and 146. Create a new pipeline that will have stages to remove outliers, as well as remove points by classification and range.  For example:

```
{
    "pipeline": [
        "./data/FoxIsland.laz",
        {
            "type": "filters.outlier",
            "method": "statistical",
            "multiplier": 3,
            "mean_k": 8
        },
        {
            "type": "filters.range",
            "limits": "Classification![135:146],Z[-10:3000]"
        },
        {
            "type": "writers.las",
            "compression": "true",
            "filename":"./data/FoxIsland_Clean.laz"
        }
    ]
}
```

- The first stage of this pipeline is removing any statistical outliers by using the [filters.outlier](https://pdal.io/en/2.5.3/stages/filters.outlier.html). For this example, we are using the statistical filtering method to compute a mean distance from each point to its nearest neighbors and compare that with a global threshold value (calculated from the mean of all distances).  There are a variety of options to adjust the calculations of the threshold and other parameters.

- The second stage of the pipeline performs a range filter.  First it includes only classes that are **NOT (note the "!")** in the range of 135 - 146.  Then it include only elevation values in the range of -10 to 3000.  

- Run the pipeline and check the classifications:

```
>> pdal info FoxIsland_Clean.laz --stats --filters.stats.dimensions=Classification  --filters.stats.count=Classification
{
  "file_size": 18100550,
  "filename": "FoxIsland_Clean.laz",
  "now": "2023-04-26T14:07:52-0600",
  "pdal_version": "2.5.3 (git-version: Release)",
  "reader": "readers.las",
  "stats":
  {
    "statistic":
    [
      {
        "average": 1.303642411,
        "count": 3073184,
        "counts":
        [
          "1.000000/2522040",
          "2.000000/474743",
          "7.000000/76401"
        ],
        "maximum": 7,
        "minimum": 1,
        "name": "Classification",
        "position": 0,
        "stddev": 0.9783966857,
        "variance": 0.9572600745
      }
    ]
  }
}
```

- Now we have a low point class (class 7), that we didn't have in the original dataset.  This is because the outlier stage writes out values as noise that are not within its statistical threshold.  So, we need to modify our pipeline to now also remove low point noise:

```
{
    "pipeline": [
        "./data/FoxIsland.laz",
        {
            "type": "filters.outlier",
            "method": "statistical",
            "multiplier": 3,
            "mean_k": 8
        },
        {
            "type": "filters.range",
            "limits": "Classification![135:146],Z[-10:3000]"
        },
	{
            "type": "filters.range",
            "limits": "Classification![7:7]"
        },	
        {
            "type": "writers.las",
            "compression": "true",
            "filename":"./data/FoxIsland_Clean.laz"
        }
    ]
}
```

- Removing the low point classification had to be added as a separate stage.  Initial attempts at doing the filtering in one stage (e.g. "limits": "Classification![135:146],Classification![7:7],Z[-10:3000]") were unsuccessful.  This may be a result of streaming mode and doing operations in memory. Adding the low point noise removal as a separate stage works fine, and now the details on the classifications show that only "ground" and "unassigned" are in the new data file: 

```
pdal info FoxIsland_Clean.laz --stats --filters.stats.dimensions=Classification  --filters.stats.count=Classification
{
  "file_size": 17452659,
  "filename": "FoxIsland_Clean.laz",
  "now": "2023-04-26T14:24:31-0600",
  "pdal_version": "2.5.3 (git-version: Release)",
  "reader": "readers.las",
  "stats":
  {
    "statistic":
    [
      {
        "average": 1.158417543,
        "count": 2996783,
        "counts":
        [
          "1.000000/2522040",
          "2.000000/474743"
        ],
        "maximum": 2,
        "minimum": 1,
        "name": "Classification",
        "position": 0,
        "stddev": 0.3651321262,
        "variance": 0.1333214696
      }
    ]
  }
}

```

- [plas.io](https://plas.io/) is a useful site to quickly look at a local las/laz file.  Loading the "clean" file that was just created, we can see that the noise has been removed:

![Fox Island Noise Removed](./images/FoxIslandNoise_Clean.png)

# Thinning
- Point cloud files can often be quite large and cumbersome to work with.  Depending on the objective, it is often useful to thin a dataset in order to make it easier and faster to work with. The [filters.sample](https://pdal.io/en/2.5.3/stages/filters.sample.html#filters-sample) utilizes a Poisson sampling to thin the dataset.

```
pdal translate ./data/FoxIsland_Clean.laz ./data/FoxIsland_Clean_Thin1m.laz sample --filters.sample.radius=1
```

![Example Thinning output](./images/Thin_Ex.png)
- Left: Dataset after thinning with 1m radius.  Right: Original dataset before thinning

- Note there are a variety of other methods to decimate data via PDAL:
    - filters.decimation
    - filters.fps
    - filters.relaxationdartthrowing 
    - filters.voxelcenternearestneighbor 
    - filters.voxelcentroidnearestneighbor
    - filters.voxeldownsize
    

# Ground Classifications & SMRF filter 
- compare using existing ground classifications vs calculating from scratch (i.e. assume a scenario where the data was provided without classifications)

# Create a DTM

 


