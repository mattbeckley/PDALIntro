# Table of Contents
1. [Acknowledgements](#acknowledgements)
2. [Background](#background)
3. [Working with PDAL](#work)
4. [PDAL mechanics](#mechanics)
5. [Inspecting Files](#inspect)
6. [Exercises](#exercises)


# Acknowledgements <a name ="acknowledgements"></a>
The focus of this workshop is getting users comfortable with basic PDAL commands, and workflows. This workshop is geared to the novice PDAL user. Content for this workshop is based on a subset of two existing PDAL workshops:
- PDAL's main workshop material: https://pdal.io/en/2.5.3/workshop/index.html
- Dr. Adam Steer's PDAL workshop from FOSS4G SotM Oceania in 2018: https://github.com/adamsteer/f4g-oceania-pdal

Users are encouraged to explore these two excellent resources for more advanced PDAL usage and examples.

# Background info on PDAL. <a name ="background"></a>
![PDAL Logo](./images/PDAL_Logo.png)

- PDAL is one of the few open source tools to work with point cloud data
- LasTools is "mostly" free, but advanced usage requires payment.
- PDAL also utilizes and aligns nicely with other open source tools such as GDAL and PROJ.
- Alignment with PROJ results in standardized coordinate system metadata


# Ways of working with PDAL <a name ="work"></a>
There are a couple of different pathways to working with PDAL: CLI, PDAL docker containers and PDAL Python API. For this workshop we will utilize the command line PDAL.

## CLI
- simplified syntax as compared to dockerized version.


```
pdal --version
pdal info --metadata ./data/OR_WizardIsland.laz
```

- Can pipe in bash commands when working with large sets of files
```
for f in *.laz;do pdal info $f --metadata >> output.txt;done
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

- Check out the excellent [python API notebook example](https://github.com/adamsteer/f4g-oceania-pdal/blob/master/notebooks/PDAL-python.ipynb) by Dr. Adam Steer for use cases and how to integrate PDAL into your python scripts via the API.

- Sample PDAL Python API commands:
```
pipeline = pdal.Pipeline(json.dumps(pipelineJson))
pipeline.validate() # check if our JSON and options were good
count = pipeline.execute() # run the pipeline
```

- Could also integrate calls to PDAL from within python scripts by using the [subprocess module](https://docs.python.org/3/library/subprocess.html)


# PDAL mechanics <a name ="mechanics"></a>

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

- Pipelines are JSON files that are written out to separate files.  To call the pipeline, specify the [pipeline](https://pdal.io/en/2.5.3/pipeline.html#) command to read in the JSON pipeline and executes the commands:

```
>> pdal pipeline pipeline_example.json
```


### Processing Modes
- PDAL has two modes of data processing: standard or stream mode.  In standard mode all input is read into memory before it is processed.  This is often necessary for operations that need access to all of the points at a given time (e.g. sorting).
- Stream mode processes data as it is read in chunks, as is much less memory-intensive.
- When using multi-stage pipelines with large datasets, there is the possibility of out-of-memory issues, especially if some of the operations require standard mode.  In these cases, it may be adviseable to break the pipeline into multiple pipelines.


# Inspecting a file <a name ="inspect"></a>
- PDAL's info tool is very powerful, and has a lot of options to help interrogate a dataset
```
pdal info ./data/FoxIsland.laz --metadata
pdal info ./data/FoxIsland.laz --schema
pdal info ./data/FoxIsland.laz --summary
```
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

## Schema
- Use the [info application](https://pdal.io/en/2.4.3/apps/info.html) with the **--schema** flag to obtain the schema for a given dataset:

```
pdal info --schema ./data/FoxIsland.laz
{
  {
  "file_size": 18337307,
  "filename": "./data/FoxIsland.laz",
  "now": "2023-04-28T12:03:38-0600",
  "pdal_version": "2.5.3 (git-version: Release)",
  "reader": "readers.las",
  "schema":
  {
    "dimensions":
    [
      {
        "name": "X",
        "size": 8,
        "type": "floating"
      },
      {
        "name": "Y",
        "size": 8,
        "type": "floating"
      },
      {
        "name": "Z",
        "size": 8,
        "type": "floating"
      },
      ...
```

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

# Exercises <a name ="exercises"></a>




    


 


