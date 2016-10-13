
# ENVI ROI to Classification (Editing in Progress)

This task creates a classification image from regions of interest (ROIs).  You may use an existing ROI file or create ROI's using the [ENVI_ImageThresholdToROI Task](http://www.harrisgeospatial.com/docs/enviimagethresholdtoroitask.html).

### Table of Contents
 * [Quickstart](#quickstart) - Get started!
 * [Runtime](#runtime) - Detailed Description of Inputs
 * [Inputs](#inputs) - Required and optional task inputs.
 * [Outputs](#outputs) - Task outputs and example contents.
 * [Advanced](#advanced) - Script performing multiple tasks in one workflow
 * [Contact Us](#contact-us)

### Quickstart

This task requires that the image has been pre-processed using the [Advanced Image Preprocessor](https://github.com/TDG-Platform/docs/blob/master/AOP_Strip_Processor.md), and that a ROI file exists or has been created.

  
	ADD Quickstart Script here


### Runtime

The following table lists all applicable runtime outputs for the ENVI ROI to Classification. An estimated Runtime for the Advanced Script example can be derived from adding the result for the two pre-processing steps. For details on the methods of testing the runtimes of the task visit the following link:(INSERT link to GBDX U page here)

  Sensor Name  |  Total Pixels  |  Total Area (k2)  |  Time(secs)  |  Time/Area k2
--------|:----------:|-----------|----------------|---------------
QB02 | 41,551,668 | 312.07 |  |  |
WV02|35,872,942 | 329.87 |  |  |
WV03|35,371,971 | 196.27 |  |  |
GE01| 57,498,000 | 332.97 |  |  |


### Inputs
The following table lists all taskname inputs.
Mandatory (optional) settings are listed as Required = True (Required = False).

  Name       |  Required  |  Valid Values       |  Description  
-------------|:-----------:|:--------------------|---------------
input_raster | True       | s3 URL, .hdr, .tiff  | Specify the input raster to apply ROIs to generate a classification image.
input_roi    | True       | ENVI format ROI file | Specify a single or an array of ROI to create the classification image from.
name         | True     |  string              | This property contains the name of the task and must be used in the workflow.

### Outputs
The following table lists all taskname outputs.
Mandatory (optional) settings are listed as Required = True (Required = False).

  Name            |  Required  |  Valid Values             | Description  
------------------|:---------: |:------------------------- |---------------
output_raster     | False      | N/A                       |This is a reference to the output classification raster of filetype ENVI.  This is a temporary file that will be deleted once the process is complete.
output_raster_uri | True       | s3 URL, .hdr, .tiff, .xml | Specify a string with the fully qualified filename and path of the output raster. If you do not specify this property, the output raster is only temporary. Once the raster has no remaining references, ENVI deletes the temporary file.


**OPTIONAL SETTINGS AND DEFINITIONS:**

Name                 |       Default    | Valid Values |   Description
---------------------|:----------------:|---------------------------------|-----------------
ignore_validate      |          N/A     |     1        |Set this property to a value of 1 to run the task, even if validation of properties fails. This is an advanced option for users who want to first set all task properties before validating whether they meet the required criteria. This property is not set by default, which means that an exception will occur if any property does not meet the required criteria for successful execution of the task.

### Advanced

Included below is a complete end-to-end workflow for ???????

	Add Advanced Task Script here......

**Data Structure for Expected Outputs:**

Your classification file will be written to the specified S3 Customer Location in the ENVI file format and tif format(e.g.  s3://gbd-customer-data/unique customer id/named directory/classification.hdr).  

For background on the development and implementation of ROI to Classification refer to the [ENVI Documentation](http://www.harrisgeospatial.com/docs/enviroitoclassificationtask.html).


###Contact Us
Document Owner - [Kathleen Johnson](kajohnso@digitalglobe.com)

> Written with [StackEdit](https://stackedit.io/).