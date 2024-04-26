.. Lumache documentation master file, created by
   sphinx-quickstart on Fri Apr  5 15:50:55 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


====================================
Heliophysics Events Registry web API
====================================

.. .. contents:: Table of Contents
..     :depth: 3

Introduction
============

.. image:: HER_arch.jpg
   :width: 600

Both Solarsoft and the `iSolSearch <https://www.lmsal.com/isolsearch>`_
web application lets users query the Heliophysics Events Registry (HER) in an interactive manner (much like Google Maps). iSolSearch interacts with HER by means of our web application programming interface (API). The API enables developers to create new third party clients for HER as well as scientists to write software code which interact with HER in an automated manner.

Querying the Heliophysics Events Registry
=========================================

Usage of an API function amounts to using HTTP to fetch a web document. The base url for all API calls is (including the question mark)::

    http://www.lmsal.com/hek/her?

Following the url base, one can append a list of keywords to specify the type of command and options for that specific command.


Simple Search for Features/Events in HER
----------------------------------------
To do a simple search for Features and Events in HER, one must append to the base url::

    cmd=search&type=column

In addition to the command (cmd), one must specify which event type to search for. For all event types, use \**::

    cmd=search&type=column&event_type=**
    
For some but not all event types, use a comma delimited list of two-letter abbreviations (see http://www.lmsal.com/hek/VOEvent_Spec.html for full list of event types). E.g. the following specifies that one wishes to search for active regions, flares and emerging flux regions::

    cmd=search&type=column&event_type=ar,fl,ef

One must also provide a time window using event_starttime and event_endtime::

    cmd=search&type=column&event_type=ar,fl,ef&event_starttime=2007-04-29T00:00:00&event_endtime=2007-05-07T00:00:00

Only entries with duration fully contained within this time will be returned.

The spatial region must also be specified. At present, we support searches by helioprojective (from Earth's perspective, in arcseconds from disk center), Stonyhurst (longitude measured from central meridian and latitude in degrees) and Carrington (longitude and latitude in degrees). Let (x1,y1) and (x2,y2) be the lower-left and upper-right coordinates of a bounding box respectively. To get everything within the disk, it suffices to specify::

    cmd=search&type=column&event_type=ar,fl,ef&event_starttime=2007-04-29T00:00:00&event_endtime=2007-05-07T00:00:00&event_coordsys=helioprojective&x1=-1200&x2=1200&y1=-1200&y2=1200
    
To specify a bounding box in Stonyhurst coordinates, give the bounding longitude and latitude in degrees. For the full sphere, do::

    cmd=search&type=column&event_type=ar,fl,ef&event_starttime=2007-04-29T00:00:00&event_endtime=2007-05-07T00:00:00&event_coordsys=stonyhurst&x1=-180&x2=180&y1=-90&y2=90
    
To get query results in XML form, set cosec=1. To get JSON, set cosec=2 (this setting is needed for iSolSearch to render the results properly).

The query for this example with JSON output would then be::

    http://www.lmsal.com/hek/her?cosec=2&cmd=search&type=column&event_type=ar,fl,ef&event_starttime=2007-04-29T00:00:00&event_endtime=2007-05-07T00:00:00&event_coordsys=helioprojective&x1=-1200&x2=1200&y1=-1200&y2=1200

To show the search results in iSolSearch, simply point your browser to the iSolSearch link with and passing the full query as a hek_query parameter. For instance, click here. Please note that iSolSearch requires cosec=2 type (JSON) format. Useful tools for inspecting JSON documents on the web are the Firefox Add-Ons JSONovich https://addons.mozilla.org/en-US/firefox/addon/10122 and JSONView https://addons.mozilla.org/en-US/firefox/addon/10869 (this one is recommended).


Reporting relations between events/features
-------------------------------------------

Relations between existing entries in HER can be reported. All relations are of the form::

    A v B 

where A and B are events/features (e.g. ARs, FLs etc) and v is a verb indicating the relation between A and B. The following verbs are allowed:

.. list-table::
   :widths: 25 50 50
   :header-rows: 1

   * - Verb
     - Meaning
     - Restrictions
   * - causes
     - There is a physical causal relation between A and B with the former as the cause of the latter.
     - 
   * - contains
     - The feature/event B is spatially and temporally contained within A.
     - Only ARs with larger (longer) spatial (temporal) extent can contain other ARs. Sunspots cannot contain other events/features.
   * - is_associated_with
     - A is associated with B.
     -
   * - splits_into
     - A splits into B
     - 
   * - merges_into
     - A merges into B
     -
   * - is_followed_by
     - A is followed by B
     - 

Users can report relations with the verbs causes and is_associated_with. For example, a researcher who has published a paper establishing the causal relationship or association between an Emerging Flux event and a Filament Eruption event can submit the relation EF causes FE. If the causal relationship was determined to be tentative, the relation EF is_associated_with FE should be used instead.

Relations using the verb contains are not to be submitted by users since HER already has the temporal and spatial information to determine/test these relationships. Since these two types of relations are rather commonplace, the cost of storage of all relations of these types will likely be too expensive (relative to testing the relations on the fly). In some case, however, it may be worth storing such relations. For example, it may be worth storing results of the complex query "Find ARs in CHs".

To report an edge, you must provide both IVORNS for A and B (labeled 'ivorn1' and 'ivorn2' in the URL), edge_type (must be a "verb" from the above list).

Optionally you may provide edge_strength (a real number in [0,1]). You may omit it (assumes a strength of 1, which is how the splits, merges, follows relations from automated methods are generally done).

Examples::

    https://www.lmsal.com/hek/her/heks?cosec=2&cmd=create_edge&ivorn1=ivo://helio-informatics.org/AR_SPoCA_20140508_034716_20140508T032848_5&ivorn2=ivo://helio-informatics.org/AR_SPoCA_20140508_074820_20140508T072848_4&edge_type=is_followed_by&edge_strength=0.9

or using curl:: 

    curl -k -b /tmp/cookiejar.txt -d cmd=create_edge -d cosec=2 -d "ivorn1=ivo://helio-informatics.org/AR_SPoCA_20140508_034716_20140508T032848_5&ivorn2=ivo://helio-informatics.org/AR_SPoCA_20140508_074820_20140508T072848_4&edge_type=is_followed_by" https://www.lmsal.com/hek/her/heks 

