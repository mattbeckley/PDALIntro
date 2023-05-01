# Table of Contents
1. [Introduction](#intro)
2. [Data Access](#access)


# Introduction <a name ="intro"></a>

## USGS 3DEP program
![USGS Logo](./images/USGSLogo.png)

- The U.S. Geological Survey’s (USGS) 3D Elevation Program (3DEP) is an ambitious effort to complete acquisition of nationwide lidar (IfSAR in AK) by 2023 to provide the first-ever national baseline of consistent high-resolution elevation data.  USGS migrated these data into the cloud, led by [HOBU](https://hobu.co/), Inc. and the U.S Army Corps of Engineers (USACE) Cold Regions Research and Engineering Laboratory (CRREL) in collaboration with [AWS Public Dataset Program](https://aws.amazon.com/opendata/?wwps-cards.sort-by=item.additionalFields.sortDate&wwps-cards.sort-order=desc).  As part of this effort the 3DEP data were standardized and written out to a [publicly-accessible, free AWS bucket in the entwine format](https://registry.opendata.aws/usgs-lidar/).

## Entwine
![Entwine Logo](./images/EntwineLogo.png)

- [Entwine](https://entwine.io/en/latest/index.html) is open source software from [HOBU, Inc.](https://hobu.co/) that organizes massive point cloud collections into streamable data services.  Entwine builds are completely lossless, so no points, metadata, or precision will be discarded even for terabyte-scale datasets.  The output format produced by entwine is the [Entwine Point Tile (EPT)](https://entwine.io/en/latest/entwine-point-tile.html), which is a simple and flexible octree-based storage format for point cloud data.  This format enables better performance when dealing with web map applications, or cloud computing with large point cloud datasets.


# Data Access <a name ="access"></a>
![USGS 3DEP Coverage](./images/USGS3DEPCoverage_OT.png)

- PDAL provides an [entwine reader](https://pdal.io/en/2.5.3/stages/readers.ept.html#readers-ept) that can easily read the USGS 3DEP data in EPT format. With a simple pipeline, it is fairly straightforward to subset the data from the AWS entwine bucket for a given dataset.  

- For example, to grab USGS 3DEP lidar data of the Statue of Liberty:

```
{
  "pipeline": [
        {
          "type": "readers.ept",
          "filename": "ept://https://s3-us-west-2.amazonaws.com/usgs-lidar-public/NY_NewYorkCity",
          "bounds": "([-8242666.7146411305, -8242531.114082908], [4966542.613156153, 4966683.155298185])"
	},
      "./data/StatueLiberty.laz"
  ]
}
```

## Dataset Names
- One of the trickier parts to working with these datasets is actually knowing the name of the dataset. There are a couple of ways of determining this:
   1.  Use the [USGS Entwine site](https://usgs.entwine.io/).  There is a search box to help subset the datasets, as well as access to cesium and potree web visualizations of the datasets.

![Entwine IO Site](./images/EntwineIO_Site.png)

   2. Using [AWS CLI tools](https://aws.amazon.com/cli/), you can can a listing of the datasets from the command line by doing:
   ```
   aws s3 ls --no-sign-request s3://usgs-lidar-public/
   ```

   3. Use [OpenTopography](https://portal.opentopography.org/dataCatalog?group=usgs) to get a listing. **Note** we remove the underscores from the names, so these need to be re-added to the EPT URL in the pipeline.
   
   4. The USGS has been migrating the metadata to AWS.  There is a [listing of 3DEP projects](http://prdtnm.s3.amazonaws.com/index.html?prefix=StagedProducts/Elevation/LPC/Projects/), but use caution.  Technically, the entwine AWS bucket is a separate project, and the processing schedule may be different than the "user-pays" bucket.  This can result in data existing in one bucket, but not the other.  Also, on rare occasions, there could be discrepancies in dataset naming conventions that can cause issues.  However, overall it is a good resource to find out more about each of the datasets.

## EPT Filename Convention

```
{
  "pipeline": [
        {
          "type": "readers.ept",
          "filename": "https://s3-us-west-2.amazonaws.com/usgs-lidar-public/NY_NewYorkCity/ept.json",
          "bounds": "([-8242666.7146411305, -8242531.114082908], [4966542.613156153, 4966683.155298185])"
	},
      "./data/StatueLiberty.laz"
  ]
}
```


# Exercises <a name ="exercises"></a>
- Try to download data directly from the AWS entwine bucket for an area of your choice.  
- If available, work through the [OpenTopography notebook: 01_3DEP_Generate_DEM_User_AOI.ipynb](https://github.com/OpenTopography/OT_3DEP_Workflows/blob/main/notebooks/01_3DEP_Generate_DEM_User_AOI.ipynb) to generate a DEM from the USGS 3DEP data.