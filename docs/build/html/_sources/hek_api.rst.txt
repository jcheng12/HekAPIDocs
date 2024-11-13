.. Lumache documentation master file, created by
   sphinx-quickstart on Fri Apr  5 15:50:55 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

===================================================
Heliophysics Events Knowledgebase API Documentation
===================================================

.. .. contents:: Table of Contents
..     :depth: 3

Introduction
============

The Heliophysics Events Knowledgebase (HEK) is a database of solar observations and events. This document describes much of the search API for the web service. The HEK comprises the Heliophysics Coverage Registry (HCR) and Event Registry (HER). HCR contains records of instrument observations (abbreviated “obs” frequently here and in the linked documents). 

This covers the search URL and webservice for the combined HCR + HER registries based at http://www.lmsal.com/hek/hcr?cmd=search-events-corr and is the main API used by http://www.lmsal.com/heksearch/ and http://iris.lmsal.com/search/. See http://www.lmsal.com/hek/software/doc/HEKSearchToolNotes.pdf and http://www.lmsal.com/hek/software/doc/IRISSearchToolNotes.pdf for guides on the graphical search tools. This document is intended for groups that want to build their own search clients, general users looking for data should probably just use the graphical tools. (Developers should play around with the graphical tools too a lot of the content can be more intuitively grasped from use then refer to this document for details on particular terms/fields. And using the “export” links can clarify the syntax). The API focuses on the HCR portion, the HER portion has an established site at http://www.lmsal.com/hek/api.html.


Return Type Parameters
======================

Outformat 
---------
Controls how the results of the query are presented; supported values:

A. **count:** returns simply the number of observations (of the primary instrument). IE “outputformat=count”. 
B. **json:** The primary output format, returning json lists. A list of common keywords with some definition is an appendix to this document. 
C. **jsonMedium**: same structure as ‘json’ but smaller number of fields for each result returned. 
D. **jsonSmall/jsonLight**: same again, the smallest collection of fields. Both this and jsonMedium were written specifically for speeding up the graphical search engines; would discourage use in SSW or other clients getting data for analysis. 

“getdistinct” : Takes as a value a single search term name, which must be of “multior” type (see below). When this is used, instead of returning full list of results, it returns a set of all values of that search term (also the “outputformat” parameter is ignored). This can be used with any combination of filters, so omitting other conditions can get all values over the whole mission, or only values matching some specific conditions. The heksearch/IRIS search web applications use this to populate and update the dropdowns of some specific fields such as the obsId and target.

Example:
http://lmsal.com/hek/hcr?cmd=search-events-corr&outputformat=jsonMedium&startTime=2015-12-02T00:00&stopTime=2016-02-23T00:00&instrument=SOTSP&getdistinct=target 


Selection Parameters
====================

All other URL parameters govern the selection of which results to present. In general, the search is a logical AND on each filter provided. 

Instrument
----------
Details the primary instrument of the search. We support IRIS and the four Hinode instruments (SOT, SOTSP, EIS, XRT). 

Instrument-specific filters/terms
---------------------------------
The SearchConfig spreadsheet (`HCRSearchConfigForAPI.pdf <https://www.lmsal.com/hpkb/doc?cmd=dcur&proj_num=HPKB00032&file_type=pdf>`__) has a list of all the supported instrument-specific search filters. These largely work on attributes of the observation like field of view, type of images taken, etc., but also include some of the planner annotations. This outline is a more detailed guide to understanding the columns of the filter configuration.

#. **inUrlName** – text tag in a URL client uses. All of these filters are specified as URL parameters. An example URL segment with four filters is &obsId=3620167602&maxsjiFOVY=80&mincadMeanPlannedSJI1400=10&hasData=true
#. **SearchCriteria** – how values are applied 

   * Minmax – numeric values. ‘min’ and ‘max’ are appended to the name in column A, so a search on HPC-X coordinate could be “&minXcen=200&maxXcen=700”. Of course, it is valid to provide only one of the min or max as well.
   * Multior – can be numeric or text. Can provide one or more values (multiple values in comma separated), such as obswheel4=0,20000,100000; the filter performs an OR search on each value provided. Note the “getdistinct=<column name>” can be used to get possible values for these columns (see above)
   * “Special” – a handful of filters work differently than either min/max or multior, they are explained below.

#. **Observation or Group**: If Observation, the filter is an attribute of the whole HCR entry/”observation”; if Group it’s a property of a specific Group. Groups are subsets of operations corresponding to a particular data type, most commonly for all images taken with a particular filter or wavelength. HCR observations can have any number (including zero, although that’s a small minority of our records) groups. 
#. The tag that the data used by this filter is returned as. 
  
   * Name of the relevant group if it’s a Group filter
   * Corresponding FITS keyword for some terms (has no functional significance to search API, but informationally useful for users looking at the data).
   * Type of value. Booleans should be, for example, &hasData=true or &hasData=false.
   * Instrument(s) filter applies to. A large number of these are IRIS-only. 
  

More detail on the working of “special” terms
---------------------------------------------
**“startTime”** & **“stopTime”** are required on every search. API returns results spanning any portion of the interval defined, so if stopTime=2013-10-11T00:00 then an observation from 2013-10-10T23:00 to 2013-10-11T01:00 is returned.
**“obsDesc”** does a free text search on the planner’s description & other related fields including some text describing the observation program.
**“obsDescEIS”** & **“obsDescSOT”** are similar, because of different planning systems the set of fields referenced for each for these two instruments are slightly different. 
**“hasdata”** refers to the compressed, science-quality level2 data for IRIS. Typically this final data production lags 3-5 days behind the quicklook events.
**“specWindows”** – A list of line names within the IRIS spectral ranges. Supply a comma-separated list and the search tool finds observations containing all of them (a logical AND on multiple window types). 

The supported names are: (note this search tool order had a more-common and less-common group, that’s why not strictly in wavelength order)::

  "Mg II k 2796", "Mg II h 2803", "C II 1335", "C II 1336", "Fe XII 1349", "C I 1354", "Fe XXI 1354", "O I 1356", C I 1356", Si IV 1394", "O IV 1400",	"O IV 1401", "Si IV 1403", 	"S I 1334","Ni II 1335", "O IV 1339", Ni II 1340", "Ni II 1346", "Cl I 1352", "C I 1357", "C I 1358", "O I 1359", "Fe II 1392", "S I 1393", "Fe II 1393", "Ni II 1393", "S I 1396", "O IV 1397", "S IV 1398", "Ni II 1399", "Fe II 1400", "S I 1402", "S IV 1405", "O IV 1405", "Fe II 1406", S IV 1406"};

**“herEvents”** – takes a comma-separated list of the two letter event codes from the Heliophysics Events Registry (HER) (see http://www.lmsal.com/hek/VOEvent_Spec.html for a listing of the abbreviations). Besides those, supports “c_fl”, “m_fl” and “x_fl” to denote flares of at least C1/M1/X1 GOES class. This geometry definition is true if there is any overlap between the observation and the event. 

**“annotated”** –for IRIS observations only. If “true”, returns only IRIS events that have been annotated, which are generally the highlights of the week they were planned on. 

**“hasSdoCubes”**, **”hasSotCubes”**, **”hasCruiser”** – if set to “true”, returns only observations with the corresponding auxiliary data product. Only for IRIS obs. See ITN 32 on http://iris.lmsal.com/documents.html for more information on those products.

**“hasBBSO”** – for a limited subset of IRIS obs coordinated with the Big Bear Solar Observatory, if “true” returns only those IRIS obs. They will have a link to a BBSO page for the corresponding ground-based observation summary.

**“hasDataALMA”**

**“hasSSTCubes” – if “true” returns.** Note that while IRIS performs many obervations each year in coordination with SST, only a small number (16?) of SST datasets have been processed for this 

**optionalcorr=**, **requiredcorr=** - These are the main parameters to specify joint searches between instruments. By “correlated” we mean observations from different instruments must have some overlap in space and time. 

Under the current system, your specific filters apply to the “primary” instrument and no additional filters on the correlated instruments. We may extend the system. 
If you have this segment in your search URL::

  instrument=IRIS&optionalcorr=SOTSP&requiredcorr=EIS

The API will look for IRIS obs which each must have an EIS obs overlapping. The data for these EIS obs is returned as well as the IRIS obs. If any SOTSP obs also overlap the IRIS results, those are included as well. Users can repeat the optionalcorr and requirecorr parameters multiple times to cover all the supported instruments. 

HOW OVERLAPPING OBSERVATIONS ARE COMPUTED
-----------------------------------------
#. All entries (both HCR observations, HER events/features) get a HPC geometry polygon.

   * For HER events, the chaincode is used if provided. If not, the bounding box is used.
   * HER events can be submitted in HRC, HGS, HGC as well but all coordinate/spatial fields are converted to HPC on ingest; that is what is used.
   * Some HER flare providers do not give a boundbox or extent. For these events a boundbox is estimated with size based on the GOES class. 
   * For HCR events, all observations get a field of view definition. For IRIS it is the ‘total FOV’ – SJI readout size combined with raster motion. Also the roll angle is applied for IRIS if it was rolled. For Hinode SOTFG and XRT events with multiple fields of view in different modes, the union is used (but, for XRT we exclude the full disk modes). 
   * If solar rotation tracking was enabled in the observation, compute the observation center HPC coordinates at the observation endtime using SRT formula. Create a second boundbox based on same FOV and rotated center point. Then the geometry becomes a polygon using the “convex hull” – smallest convex polygon where all points are on boundary or contained within. 

#. Scan over all HCR-HCR pairs, and HER-HCR pairs. For the time being we do not support overlapping event-event searches. For each pair of entries A, B (where A is either HCR or HER, B is HCR):

   *	All entries have a start and end time, check if the time ranges of A, B overlap.
   * If so, check if geometry overlaps between A, B
   * If both types overlap, compute a geometry score defined as the larger of  (area of intersection of A, B ) / (area of A) and (area of intersection) / (area of B). The areas are in HPC square arcsec, not projected.
   * Compute time score in analogous fashion. 
   * When storing the correlation, also check if the event coo. Note that for observations this point is always the center of the boundbox, but for HER events there are some automated feature finding algorithms that provide chaincodes and the event point is not necessarily the center of the chaincode. 

#. The geometries from step 1 for all of these pairs are also compared, and overlapping regions computed. 

Note that there are approximations made, particularly with certain flare sizes, and when two entries have significantly different start times with rotation tracking there may be false positives. We provide this as a data discovery tool; scientific users should co-align the data themselves if they are doing detailed analysis. Also the current implementation does not apply SRT to HER entries currently but will in the future. (The “events” in the HER are largely short enough that it would be minor, but there are “feature” entries for AR, CH,SS that last several hours to a day and would be affected significantly.)  In the future we may also transform geometries to Carrington coordinates where possible to make the rotation more precise. Event providers to HER may follow different standards of event durations. In particular this is an issue for flares, different modules may report different start and (especially) end times, which can affect whether an overlap is detected.


Examining search results
========================

This section gives a brief explanation of our JSON return format (please see https://www.json.org/ if unfamiliar with JSON). The two main nestings in our results are “Events” and “Groups” (see the notes for column C above). Both HCR observations and HER events are labeled “Events” in the JSON. Only HCR observations can have Groups. 

List of common json keys for HCR:
Presenting this list with example values. The HER return format is similar and is at https://hekapidocs.readthedocs.io/en/latest/. In the future there will be a linked document with an exhaustive list of all keys but these are the most common/useful. 

The unique identifier in the HCR is the “eventId”, which is a IVORN (see http://wiki.ivoa.net/twiki/bin/view/IVOA/ResourceNameSemantics). Generally our IVORNS are made from the instrument and start time of the observation:: 

  "eventId": "ivo://sot.lmsal.com/VOEvent#VOEvent_IRIS_20131014_113941_ 3860257107_2013-10-14T11:39:412013-10-14T11:39:41.xml"

The corresponding event summary page will be found at 
http://www.lmsal.com/hek/hcr?cmd=view-event&event-id=<value of “eventId”>

However the “://” in the IVORN part of that URL, as well as the ‘#’, often needs escaping, such as this (browsers may not strictly need to escape the ‘://’ but may do it anyways)::

  http://www.lmsal.com/hek/hcr?cmd=view-event&event-id=ivo%3A%2F%2Fsot.lmsal.com%2FVOEvent%23VOEvent_IRIS_20131014_113941_3860257107_2013-10-14T11:39:412013-10-14T11:39:41.xml

**“eventKey”: 1234575** – this is unique integer ID of the event, used as the key in correlations.

**“instrument”**, **“startTime”**, **“stopTime”** – same as in the search filters.

**“xCen”: “105”**, **“yCen”: “-400”** – These give the coordinates of the center of the observation at its start time, in HPC arcseconds. 

**“xFov”: “95.0”**, **“yFov” : “66.0”** – gives field of view, also in HPC-arcseconds. Note these are in the groups as well – some HCR instruments take different types of data with different fields of view. 

**“raster_fovx”**, **“raster_fovy”**, **“sji_fovx”**, **“sji_fovy”** – for IRIS, give specific breakdown of the field of view. The overall field of view sums raster + sji in the X direction but uses just one of the Y direction (for all IRIS non-calibration obs the two fovy’s should be the same). 

**“roll_angle” : “-90”** - also IRIS only, roll at time of observation. In degrees, IRIS can roll from -90 to +90 deg.

**“num_images” : “300”** – number of images in a group. 

**“cadence_avg_asplanned” : “20.5”** – for IRIS, time between images in a group (in seconds). 

**“target” : “QS”** – planner’s target for the observation. Besides the event codes in the HER, we use “QS” for quiet sun.

*“umodes” : "XRT Open Gband, XRT Al_poly Open, 256 x 256, 512 x 512"* - For Hinode instruments, a list of group names. 

Correlation Results in JSON
---------------------------
If using the optionalcorr and/or requiredcorr parameters, after the set of all Events in the result there will be another set::

  “Correlations": [
    {
      "key1": 3331467,
      "time_overlap": 1,
      "key2": 3340833,
      "score": 0.3057028254999372,
      "type2": "XRT",
      "reg1": "HCR",
      "ivorn2": "ivo://sot.lmsal.com/VOEvent#VOEvent_ObsX2018-05-15T17:28:30.000.xml",
      "reg2": "HCR",
      "ivorn1": "ivo://sot.lmsal.com/VOEvent#VOEvent_IRIS_20180515_170318_3640106077_2018-05-15T17:03:182018-05-15T17:03:18.xml",
      "type1": "IRIS",
      "spatial_overlap": 0.9760187660508767}, 
      {
      "key1": 8408656,
      "key2": 3301566,
      "score": 1,
      "type2": "IRIS",
      "reg1": "HER",
      "ivorn2": "ivo://sot.lmsal.com/VOEvent#VOEvent_IRIS_20180428_184020_3690015104_2018-04-28T18:40:202018-04-28T18:40:20.xml",
      "reg2": "HCR",
      "ivorn1": "ivo://helio-informatics.org/ER_IRIS_SG_C_II_KathyReeves_20180430_185912",
      "type1": "ER"
    },
  ]

This denotes an edge between two entries in the Events results. The “key” and “ivorn” tags ID which events are involved, “reg” and “type” denote whether HCR/HER and either Instrument (HCR) or Event Type (HER). Overlaps and scores show how good of a spatial match they are. 


Other notes
===========
Results currently limited to 400 per query on full searches, but can go higher if using outputformats jsonLight/jsonMedium. If doing correlated searches, the number of “rows” will go down as the limit is shared across the instruments. 

Typically, about 11 observations are done per day on IRIS; as of October 2024 IRIS is over 44,000 as it recently passed 11 years on in operation. Hinode instruments typically have a bit fewer per day.

HER events search
-----------------

http://www.lmsal.com/hek/hcr?cosec=2&cmd=search-her&type=column

As part of the work to allow joint observation/event searches, the HER API functionality was ported to the HCR. Goal was to leave the original HER event search unchanged as the new one is enhanced (We did not want to change the legacy ISolSearch webpage or the SSW API). The main format is identical to that of the current HER API based at http://www.lmsal.com/hek/her?cosec=2&cmd=search&type=column& (Again, there are legacy clients we wanted to leave unchanged, as some of them like the SunPy API we do not have control over). That API is described at https://hekapidocs.readthedocs.io/en/latest/ 

Compared to the old HER format, the new one takes additional parameters:
optionalcorr, requiredcorr – taking HCR instrument classes and adding coaligned observations.  Works analogously to how observation-observation correlations are done & described above.

Other HCR Commands for observation viewing
------------------------------------------

View recent summaries: http://www.lmsal.com/hek/hcr?cmd=view-recent-events&instrument=iris 

View obs summary page (either planning or observation) with example IVORN: http://www.lmsal.com/hek/hcr?cmd=view-event&event-id=ivo%3A%2F%2Fsot.lmsal.com%2FVOEvent%23VOEvent_IRIS_20180119_133521_3610108077_2018-01-19T13%3A35%3A212018-01-19T13%3A35%3A21.xml 

Get JSON for an individual obs (IRIS only): http://www.lmsal.com/hek/hcr?cmd=json-for-event&event-id=ivo%3A%2F%2Fsot.lmsal.com%2FVOEvent%23VOEvent_IRIS_20180119_133521_3610108077_2018-01-19T13%3A35%3A212018-01-19T13%3A35%3A21.xml 


Previous HCR APIs
=================

Old IRIS API
------------
*http://www.lmsal.com/hek/hcr?cmd=search-events3&*

This is still used by http://iris.lmsal.com/search/. It was the early version of this newer API and will remain live for the time being to support SSW client methods; As of July 2017, the functionality should be identical to ‘search-events-corr’ with the following choices:

1.	Instrument = IRIS (cannot be changed, IRIS only, also implicit in search-events3)
2.	No correlated instrument searches. 

Legacy HCR API
--------------
http://www.lmsal.com/hek/hcr?cmd=submit-search-events2
Basis of old graphical search tool at http://www.lmsal.com/hek/hcr?cmd=search-events2 

Described at https://www.lmsal.com/sdodocs/doc?cmd=dcur&proj_num=SDOD0046&file_type=txt 

This is the original LMSAL search tool, developed for the Solar-B/Hinode mission but does support all instruments in the HCR. In particular one must explicitly provide ‘select=<term>’ for each column/term. It does not allow any correlated observation/events searches. 