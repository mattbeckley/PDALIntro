# PDAL Independent Exploration

For the final ~1.25 hours of the workshop, participants will be asked to take what they've learned during the day, and to use PDAL to explore building a processing workflow. This is a chance to internalize what you've learned, and to think about how you'd apply PDAL to your specific datasets, application, or reseach objective.

As we near the end of the exploration session, we'll ask you to upload a couple of screen captures / images / brief summary of what you did, and what you learned. Show us pretty pictures, successes, failures, problems, challenges, etc. The point is to share with each other and learn from each other's experiences.

- [Upload your explorations HERE](https://drive.google.com/drive/folders/1xxtLaG15HNpA0CR8q81_snCckFDB9ubv?usp=sharing) (please put your name in the file name)

# Need Ideas?

- Revisit some of the exercises from the previous tutorials. Run them on new/different datasets that are of interest to you.
- - Do a comparison of ground classifications for a dataset of your choice:
   - Create a ground-classified DTM from vendor-provided ground classifications (class=2)
   - Create a custom ground-classified DTM from scratch
   - Use GDAL raster math to difference the two DTMs so you can see the effect on representatio of bare earth.
   - Visualize the DTMs and difference grids in QGIS.
- Create a Canopy Height Model from a lidar point cloud for an area of interest to you. Try the [filters.hag_delaunay](https://pdal.io/en/latest/stages/filters.html) to classify a point cloud by height above ground (hint: you need to first classify the ground).
- Dig into the PDAL docs and try a filter or application that looks interesting that we haven'y explored today.
- Explore USGS 3DEP lidar accessed via PDAL - see [AccessUSGS3DEPEntwine.md](https://github.com/mattbeckley/PDALIntro/blob/main/AccessUSGS3DEPEntwine.md) and/or the notebooks avaialble here: [Reproducible scientific workflows for accessing, processing, and visualizing USGS 3DEP lidar data](https://github.com/OpenTopography/OT_3DEP_Workflows).
