Shumagin files from Glenn Nelson

This is a collection of programs for the Shumagin Gap study.
These programs focus on obtaining daily records from the seismometers,
then using EqCorrScan to find repeating events.
QuakeML files are event catalogs with picks. For convenience I generate
CSV files of event hypocenters and append columns that are used to organize
these events for further study.

I fetched the earthquake catalog for the Shumagin zone for the period from
June 2018 to end of August 2019. These events are filtered to a max depth of 50 km,
minimum magnitude of 2.5. They are from NCEDC? The file is quakes.csv.
I used the USGS subduction zone slab model for Alaska. This is NetCDF4 files.
The file alu_slab2_dep_02.23.18.grd represents the depth of the top interface
as a grid in lat-long coordinates. Evidently it can be plotted as a TIFF image, but
is more useful when considered as a 3D grid.
Python program ev_slab_filter.py reads quakes.csv using Pandas and reads slab.grdfile
using netcdf. Each earthquake hypocenter is compared with the slab depth. The hypocenter
depth error is used to decide if the event depth is "close" to the slab depth. The catalog is 
rewritten with two additional columns for each event: dep_diff and ok_dep.
dep_diff is (hypocenter depth) - (slab depth).
ok_dep is TRUE if dep_diff is less than depthError.
The new output file is named slab_events.csv.
For convenience, two additional catalogs are written: slab_true.csv and slab_false.csv.
These files contain only TRUE or FALSE depth events.

Script gmt_slab.bat is a Windows script that uses GMT to plot both slab events and
the USGS slab model. The same color code is used for both. This script can run as
a bash script in Linux.

Program get_trace.py reads mseed files and searches for events that are defined by
time and space origin. The program inputs are station name (determines which mseed file)
and a CSV catalog file. All events in the catalog are searched for in the mssed file.
For each event, the distance to the station and approximate P arrival time is computed.
When event is found, a trace is read and written to a database (maybe).
Since the mseed files and event catalogs are ordered from oldest to most recent,
the files are always read sequentially, i.e., no backing up required.

I'm having funny problems.shum_eqcorrscan.py uses local quakeml file and local streams.
This program is based on parkfield_ex411.py, which I cribbed from eqcorrscan examples.
First I encounter a problem with Party object:
    # GN: error here, because there is no member "families"
    #family = sorted(party.families, key=lambda f: len(f))[-1]
    family = party[0]
But in parkfield example it does have 'families'. In the code the only difference is
that parkfield uses party=tribe.client_detect and I use party=tribe.detect. But maybe
families is missing for another reason?
And then I have a problem with Family:
    # GN: error here, because 'Detection' object has no attribute 'template'
    #fig = family.template.st.plot(equal_scale=False, size=(800, 600))
    fig = family.st.plot(equal_scale=False, size=(800, 600))
It also has no member 'st'.
But the parkfield example does have family.template.
Now I'm trying to plot as such:
    #streams = family.extract_streams(stream=st, length=10, prepick=2.5)
    streams = family.extract_stream(stream=st, length=10, prepick=2.5)
But I don't even have the extract_streams method! Only have extract_stream!

Now I have another problem. At least one of 3 traces that I create with get_trace_48
is coming up short. Tribe.construct() has param process_len=86400. All stream traces
must be at least 80% of this length or it aborts.
So I set process_len=64800 (75%), but it still complains about 86400. I have looked at source
for tribe.py and template_gen.py and I can see that everywhere the default process_len=86400.
So the override is failing somewhere in this chain.


