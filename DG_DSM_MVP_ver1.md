﻿## DG Digital Surface Model MVP 
(Adam will change the name to something he likes)

This document describes the end-to-end application of the Satellite Stereo Pipeline (satellite_stereo_pipeline) and DSM Sweep (dsm_Sweep) Tasks to produce a digital surface model (DSM) from in-track stereo pair images.

#### TABLE OF CONTENTS

* [Quickstart](#quickstart) - Get started!
 * [Inputs](#inputs) - Required and optional task inputs.
 * [Outputs](#outputs) - Task outputs and example contents.
 * [Accuracy Estimates](#accuracy-esimates) -
 * [Known Issues](#known-issues) - 
 * [Contact Us ](#contact-us) - Contact information.

#### IMAGERY EXAMPLES

ADD IMAGES AND ANIMATIONS

#### QUICKSTART TUTORIALS

This script uses the combined task of the DG Stereo Solution to produce a digital surface model (DSM) from an image stereo pair.  When the input raster is larger than 3000 x 3000 pixels, satellite_stereo_pipeline trims by 1000 pixels on each side.   This prevents expensive and slow post processing along collect boundaries that for full stereo pairs are poorly resolved.

```python
# Test Run for Satellite Stereo Pipeline & DSM Sweep

from gbdxtools import Interface
gbdx = Interface()

# Run the Satellite Stereo Pipeline and DSM_Sweep Tasks end to end.  
# You must stage the data to your customer S3 bucket

s2ptask = gbdx.Task('satellite_stereo_pipeline',
	input_pair='s3://gbd-customer-data/full path to customer's S3 location/input_pair/'
)

sweeptask = gbdx.Task('dsm_sweep', 
	dsm_input = s2ptask.outputs.data,
	exterior_shapefile = 's3://gbd-customer-data/full path to customer's S3 location/exterior_shapefile/',
	ortho = 's3://gbd-customer-data/full path to customer's S3 location/ortho/',
	truth_dsm = 's3://gbd-customer-data/full path to customer's S3 location/truth_dsm/'
)

workflow = gbdx.Workflow([s2ptask,sweeptask])

#Edit the following line(s) to reflect specific folder(s) for the output file.  The final desitnation directory names must be 'data'.

workflow.savedata(s2ptask.outputs.data, location='customer s3 output location/s2p_out/data/')
workflow.savedata(sweeptask.outputs.data, location='customer s3 output location/sweep_out/data/')

workflow.execute()
print workflow.id
print workflow.status
```


#### INPUT OPTIONS

### For satellite_stereo_pipeline Task:

The `input_pair` is the only required data.  You can run a smaller test subset of the input data by setting the x-y pixel values for the upper left corner of the subset region and setting the height and width of the region of interest.   Default Input types have set optimal values.  Depending on feedback, future versions of the algorithm will allow the customer to change these values.

Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
input_pair       |  YES   |  directory  | S3 directory location of input data.
subset_ul_row    |  NO    |  string     | upper left y-pixel value for subset
subset_ul_col    |  NO    |  string     | upper left x-pixel value for subset
subset_height    |  NO    |  string     |   pixel height of subset
subset_width     |  NO    |  string     |   pixel width of subset
max_processes    |  NO    |  preset    | default setting is: 16   ; cannot be changed by user
tile_size        |  NO    |  preset    | default setting is: 500   ; cannot be changed by user
matching_algorithm |  NO  |  preset    | default setting is: MGM   ; cannot be changed by user

### For dsm_sweep Task:
Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
dsm_input            |  YES   |  directory  | S3 directory location of input  data. Use: dsm_input = task.outputs.data as shown in the example script.
exterior_shapefile   |  NO    |  directory  | S3 directory location of order shapefile
gcp_shapefile        |  NO    |  directory  | S3 directory location of gcp shapefile
ortho  |  NO  |  directory  | S3 directory location of orthorectified image with same footprint
truth_dsm     |  NO  |directory  |  S3 directory location of elevation truth data set		     		                		 
water_masking  | NO  | preset  |     default setting is: true ; cannot be changed by user                      
hole_filling   |  NO  | preset |   default setting is: vsr  ; cannot be changed by user                               
hole_filling_max_search 	| NO  | preset| default setting is: 50  ; cannot be changed by user	
vsr_guide      |  NO  |  preset  | default setting is: minval  ; cannot be changed by user                                       
vsr_prior_prob |  NO  |  preset  | default setting is: 0.1 ; cannot be changed by user	
filter_spikes  |  NO  |  preset  | default setting is: true ; cannot be changed by user
exterior_buffer |  NO  |  preset | default setting is: 1000 ;  see known issues					


#### OUTPUTS

### For satellite_stereo_pipeline Task:

Name             |     Required    |   Input Type   |   Description
-----------------|:---------------:|------------------|--------
data             |      YES    |  directory           | S3 directory location of output data.

If you choose to save the raw digital surface model results, this is the expected contents of the `data directory`.  The dsm.tif is the output file containing the raw digital surface model results from the `satellite_stereo_pipeline` Task.

```
	4	 yourImage.RPC	
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

Below are the expected contents of the `data directory` for the final dsm product.  The dsm.tif is the output file containing the adjusted digital surface model results from the `dsm_sweep` Task.

```

	3	 dsm.tif	
	4	 dsm_sweep.json		
	5	 spike_filtered.tif	
	6	 spike_filtered.tif.aux.xml	
	7	 temp_water.cpg		
	8	 temp_water.dbf		
	9	 temp_water.prj		
	10	 temp_water.shp		
	11	 temp_water.shx		
	12	 water_mask.dbf		
	13	 water_mask.prj		
	14	 water_mask.shp		
	15	 water_mask.shx
```


#### ACCURACY ESTIMATES

Want feedback on this before I take the time to type a table.  What do we want to say here? I'm considering a table that has our best sample data Cat ID's with stats for those pairs: Salt Lake City, Melbourne, Garden Grove, Alameda.
	

#### KNOWN ISSUES

 - The `exterior_buffer` is disabled in this version.
 - Full stereo pairs (approx. 30,000 x 50,000 pixels) take about 14-18 hours to run
~ 12 hours satellite_stereo_pipeline; ~4 hours dsm_sweep.  The satellite_stereo_pipeline runs on a `raid` instance
 

#### CONTACT US


```