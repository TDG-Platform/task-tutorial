﻿## DG Digital Surface Model MVP 

This document describes the end-to-end application of the Satellite Stereo Pipeline (satellite_stereo_pipeline) and DSM Sweep (dsm_Sweep) Tasks to produce a digital surface model (DSM) from in-track stereo pair images. DSM Sweep removes spikes and fills holes that remain after the initial DSM processing by the Satellite Stereo Pipeline.  

#### TABLE OF CONTENTS

 * [Quickstart](#quickstart-tutorials) - Get started!
 * [Inputs](#input-options) - Required and optional task inputs.
 * [Outputs](#outputs) - Task outputs and example contents.
 * [Known Issues](#known-issues) 
 * [Contact Us ](#contact-us) - Contact information.

#### IMAGERY EXAMPLES

ADD IMAGES AND ANIMATIONS

#### QUICKSTART TUTORIALS

This script uses the combined task of the DG DSM MVP to produce a digital surface model (DSM) from an image stereo pair.  When the input raster is larger than 3000 x 3000 pixels, satellite_stereo_pipeline trims by 1000 pixels on each side.   This prevents expensive and slow post processing along collect boundaries that for full stereo pairs are poorly resolved.

```python
# Test Run for Satellite Stereo Pipeline & DSM Sweep

from gbdxtools import Interface
gbdx = Interface()

# Run the Satellite Stereo Pipeline and DSM_Sweep Tasks end to end.  
# You must stage the data to your customer S3 bucket.  Each input value requires a separate directory.

s2ptask = gbdx.Task('satellite_stereo_pipeline',
	input_pair='s3://gbd-customer-data/full path to customer's S3 location/input_pair/'
)

# dsm_input is the only required input value.  Edit the others according to whether those parameters are used.

sweeptask = gbdx.Task('dsm_sweep', 
	dsm_input = s2ptask.outputs.data,
	exterior_shapefile = 's3://gbd-customer-data/full path to customer's S3 location/exterior_shapefile/',
	ortho = 's3://gbd-customer-data/full path to customer's S3 location/ortho/',
	truth_dsm = 's3://gbd-customer-data/full path to customer's S3 location/truth_dsm/'
)

workflow = gbdx.Workflow([s2ptask,sweeptask])

# Edit the following line(s) to reflect specific folder(s) for the output file.  
# The final desitnation directory names must be 'data'.

workflow.savedata(s2ptask.outputs.data, location='customer s3 output location/s2p_out/data/')
workflow.savedata(sweeptask.outputs.data, location='customer s3 output location/sweep_out/data/')

workflow.execute()
print workflow.id
print workflow.status
```


#### INPUT OPTIONS

The input data consists of a pair of in-track panchromatic stereo images in tiff format and factory-ordered (or preprocessed) to Level OR2A (ortho-ready level 2A).  The Task will first look for an RPC file, and if none is present, look for an RPB file and perform the RBP to RPC conversion. The RPC/RPB files should be in the same directory as the input_pair files. Bundle-block-adjustment (BBA) is highly recommended for the most accurate results.

### For satellite_stereo_pipeline Task:

The `input_pair` is the only required data.  You can run a smaller test subset of the input data by setting the x-y pixel values for the upper left corner of the subset region and setting the height and width of the subset region.   Default Input types have set optimal values.  Depending on feedback, future versions of the algorithm will allow the customer to change these values.

Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
input_pair       |  YES   |  directory  | S3 directory location of input data.
subset_ul_row    |  NO    |  string     | upper left y-pixel value for subset
subset_ul_col    |  NO    |  string     | upper left x-pixel value for subset
subset_height    |  NO    |  string     |   pixel height of subset
subset_width     |  NO    |  string     |   pixel width of subset
max_processes    |  YES    |  preset    | default setting is: 16, which is the maximum allowed
tile_size        |  NO    |  preset    | default setting is: 500   ; should not be changed by user
matching_algorithm |  NO  |  preset    | default setting is: MGM   ; should ot be changed by user

### For dsm_sweep Task:

Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
dsm_input            |  YES   |  directory  | S3 directory location of input  data. Use: dsm_input = task.outputs.data as shown in the example script.
exterior_shapefile   |  NO    |  directory  | S3 directory location of order shapefile
gcp_shapefile        |  NO    |  directory  | S3 directory location of gcp shapefile
ortho  |  NO  |  directory  | S3 directory location of orthorectified image with same footprint; if specified, it is used by VSR to improve the DSM edges.
truth_dsm     |  NO  |directory  |  S3 directory location of elevation truth data set		     		                		 
water_masking  | NO  | preset  |     default setting is: true ; should ot be changed by user                      
hole_filling   |  NO  | preset |   default setting is: vsr  ; should ot be changed by user                               
hole_filling_max_search 	| NO  | preset| default setting is: 50  ; should ot be changed by user	
vsr_guide      |  NO  |  preset  | default setting is: minval  ; should ot be changed by user                                      
vsr_prior_prob |  NO  |  preset  | default setting is: 0.1 ; should ot be changed by user	
filter_spikes  |  NO  |  preset  | default setting is: true ; should ot be changed by user			


#### OUTPUTS

### For satellite_stereo_pipeline Task:

Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
data             |      YES    |  directory           | S3 directory location of output data.

If you choose to save the raw digital surface model results, this is the expected contents of the `data directory`.  The dsm.tif is the output file containing the raw digital surface model results from the `satellite_stereo_pipeline` Task.

```
	3	 yourImage1.RPC
	4	 yourImage2.RPC
	5	 ReturnComments.txt	
	6	 SuccessAndFailureInfo.txt	
	7	 config.json	
	8	 dsm.tif	
	9	 dsm.tif.aux.xml	
	10	 dsm.vrt	
	11	 gdalbuildvrt_input_file_list.txt	
	12	 global_pointing_pair_1.txt	
	13	 global_srcwin.txt	
	14	 s2p_public.json	
	15	 tiles.txt	
	16	 tiles/
```

### For dsm_sweep Task:

Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
data             |      YES    |  directory           | S3 directory location of output data.

Below are the expected contents of the `data directory` for the final `dsm_sweep` Task product.  The dsm.tif is the final corrected tif output with spikes removed and holes filled; dsm_filtered.tif is the spike filtered output;  holes_filled_subset.tif. There will only be a 'Statistics.txt` file if you have provided a truth dataset.

```

	3        DifferenceHistogram.png	
	4	 Statistics.txt	
	5	 dsm.tif	
	6	 dsm_filtered.tif	
	7	 dsm_minus_truth.tif	
	8	 dsm_sweep.json	
	9	 dsm_unfiltered.tif	
	10	 holes_filled.tif.aux.xml	
	11	 holes_filled_subset.tif	
	12	 holes_filled_subset.tif.aux.xml	
	13	 spike_filtered.tif.aux.xml	
	14	 temp_water.cpg	
	15	 temp_water.dbf	
	16	 temp_water.prj	
	17	 temp_water.shp	
	18	 temp_water.shx	
	19	 water_mask.dbf	
	20	 water_mask.prj	
	21	 water_mask.shp	
	22	 water_mask.shx
```
	

#### KNOWN ISSUES

 - The output DSM will be half meter resolution. The only tested input is half meter OR2As. Behavior with other resolutions is unknown.
 - The orthorectified image can be a tif file or a vrt file with the associated tif files.
 - Full stereo pairs (approx. 30,000 x 50,000 pixels) take about 14-18 hours to run
~ 12 hours satellite_stereo_pipeline; ~4 hours dsm_sweep.  The satellite_stereo_pipeline runs on a `raid` instannce.
 - These Processors can also be applied to cross-track stereo pairs, however, this application of the Tasks has not been heavily tested. Cross-track images should only be utilized when in-track stereo pairs are not available.
 

#### CONTACT US
Tech Owner: [Rick Clelland](#Richard.Clelland@digitalglobe.com) & Editor:  [Kathleen Johnson](#kathleen.johnsons@digitalglobe.com)

