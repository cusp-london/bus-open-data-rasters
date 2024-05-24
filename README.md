# Bus Open Data Rasters

This repository contains raster data captured via the [DfT Bus Open Data Service](https://www.bus-data.dft.gov.uk/).

The data is provided in GeoTIFF file format for different areas comprised of bounding boxes defined by longitude and latitude for hourly time periods at a spatial resolution of 50 metres.

There are two types of data available in raster format:

1. distinctJourneyCounts (count of journeys): includes a count of the number of distinct bus journeys (defined as a vehicle travelling along a route in a single direction) entering each grid square within the specified time period;

2. averageSpeeds (in metres per second): which takes observations every minute, links observations within a bus journey, and then computes the average speed between consecutive observations. The value in each grid square is an average of all journey speeds that intersect with the square.

Within this repository the data is provided in compressed zip format, where the name of the zip file identifies the data value, the geographic area (see the section on [Bounding Boxes](#bounding-boxes) for a full definition) and the date range included in each zipped file.

## List of data available data

### London

[averageSpeeds_London_271023to301123.zip](data/London/averageSpeeds_London_271023to301123.zip)

[averageSpeeds_London_011223to080124.zip](data/London/averageSpeeds_London_011223to080124.zip)

[averageSpeeds_London_090124to310124.zip](data/London/averageSpeeds_London_090124to310124.zip)

[averageSpeeds_London_010224to290224.zip](data/London/averageSpeeds_London_010224to290224.zip)

[averageSpeeds_London_010324to310324.zip](data/London/averageSpeeds_London_010324to310324.zip)

[averageSpeeds_London_010424to300424.zip](data/London/averageSpeeds_London_010424to300424.zip)

[averageSpeeds_London_010524to080524.zip](data/London/averageSpeeds_London_010524to080524.zip)

[distinctJourneyCounts_London_271023to080124.zip](data/London/distinctJourneyCounts_London_271023to080124.zip)

[distinctJourneyCounts_London_090124to310124.zip](data/London/distinctJourneyCounts_London_010224to290224.zip)

[distinctJourneyCounts_London_010224to290224.zip](data/London/distinctJourneyCounts_London_010224to290224.zip)

[distinctJourneyCounts_London_010324to310324.zip](data/London/distinctJourneyCounts_London_010324to310324.zip)

[distinctJourneyCounts_London_010424to300424.zip](data/London/distinctJourneyCounts_London_010424to300424.zip)

[distinctJourneyCounts_London_010524to080524.zip](data/London/distinctJourneyCounts_London_010524to080524.zip)

### North East

[averageSpeeds_NorthEast_171223to090124.zip](data/NorthEast/averageSpeeds_NorthEast_171223to090124.zip)

[averageSpeeds_NorthEast_100124to310124.zip](data/NorthEast/averageSpeeds_NorthEast_100124to310124.zip)

[averageSpeeds_NorthEast_010224to290224.zip](data/NorthEast/averageSpeeds_NorthEast_010224to290224.zip)

[averageSpeedsNorthEast_010324to310324.zip](data/NorthEast/averageSpeeds_NorthEast_010324to310324.zip)

[averageSpeedsNorthEast_010424to300424.zip](data/NorthEast/averageSpeeds_NorthEast_010424to300424.zip)

[averageSpeedsNorthEast_010524to070524.zip](data/NorthEast/averageSpeeds_NorthEast_010524to070524.zip)

[distinctJourneyCounts_NorthEast_171223to091024.zip](data/NorthEast/distinctJourneyCounts_NorthEast_171223to091224.zip)

[distinctJourneyCounts_NorthEast_100124to310124.zip](data/NorthEast/distinctJourneyCounts_NorthEast_100124to310124.zip)

[distinctJourneyCounts_NorthEast_010224to290224.zip](data/NorthEast/distinctJourneyCounts_NorthEast_010224to290224.zip)

[distinctJourneyCounts_NorthEast_010324to310324.zip](data/NorthEast/distinctJourneyCounts_NorthEast_010324to310324.zip)

[distinctJourneyCounts_NorthEast_010424to300424.zip](data/NorthEast/distinctJourneyCounts_NorthEast_010424to300424.zip)

[distinctJourneyCounts_NorthEast_010524to070524.zip](data/NorthEast/distinctJourneyCounts_NorthEast_010524to070524.zip)

## Example Usage

If using Python, it is recommended to use the [rasterio](https://rasterio.readthedocs.io/en/stable/) package to read the data.

An example script that creates a short animation of the data in London is shown below:

```python
from osgeo import gdal
import rasterio
import rasterio.plot
import matplotlib.pyplot as plt
from matplotlib import animation
import os
from dotenv import dotenv_values

config = dotenv_values(".env")
data_directory = config['DATADIR']
date_directory = data_directory + '\\distinctJourneyCounts\\2023\\10\\28\\'
visdir = config['VISDIR'] + '\\distinctJourneyCounts\\'

files = os.listdir(date_directory)
files_to_load = []
for f in files:
    files_to_load.append(f)
files_to_load.sort()

# Get max value for the files of interest
max_value = 0
for f in files_to_load:
    raster_data = rasterio.open(date_directory + f)
    file_max_value = raster_data.read(1).max()
    if file_max_value > max_value:
        max_value = file_max_value

fig = plt.figure(figsize=(4*raster_data.width/172, 4*raster_data.height/172),
                 dpi=172)
fig.subplots_adjust(left=0, bottom=0, right=1, top=1, wspace=None, hspace=None)
ax = plt.subplot(111)
raster_data = rasterio.open(date_directory + files_to_load[0])
im = plt.imshow(raster_data.read(1), cmap='hot', vmin=0, vmax=max_value)
ax.axis('off')
title = ax.text(x=0.5,
        y=0.95,
        s=files_to_load[0].split('_')[0],
        horizontalalignment='center',
        verticalalignment='center',
        transform=ax.transAxes,
        color='white',
        fontsize=12)

def animate(i):
    a = im.get_array()
    raster_data = rasterio.open(date_directory + files_to_load[i])
    im.set_data(A=raster_data.read(1))
    title.set_text(files_to_load[i].split('_')[0])
    return im,

anim = animation.FuncAnimation(fig, animate, frames=24, interval=20, blit=True)
anim.save(visdir + 'distinctJourneyCounts_mplanimate_20231028.gif',
          fps=1)
```

This produces the following visualisation:

![Example visualisation of bus open data rasters](examples/distinctJourneyCounts_mplanimate_20231028.gif)

## Time frames

Data are currently available at an hourly level of resolution from 27th October 2023 to 8th May 2024 in the case of London and from 17th December 2023 to 7th May 2024 in the case of the bounding box in the North East.

## Bounding boxes

In the initial release, there are two areas of interest: one containing the Inner London boroughs of Camden, Greenwich, Hackney, Hammersmith and Fulham, Islington, Kensington and Chelsea, Lambeth, Lewisham, Newham, Southwark, Tower Hamlets, Wandsworth and Westminster; and an area within the Northeast of England, containing Newcastle upon Tyne, Sunderland, Gateshead and Durham.

The Inner London bounding box is defined by the following two Latitude/Longitude coordinates:

```
(-0.260707, 51.412938)
(0.128712, 51.574489)
```

The Northeast bounding box is defined by the following two Latitude/Longitude coordinates:

```
(-1.9911, 54.7291)
(-1.2777,55.1888)
```

## Contributors

This dataset is developed by [Peter Baudains](https://github.com/peterbaudains) at [CUSP London](https://cusplondon.ac.uk/).

The source data is provided by the Department for Transport via the [Bus Open Data Service](https://www.bus-data.dft.gov.uk/).

## License Information

Data contains public sector information licensed under the [Open Government License v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).
