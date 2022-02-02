---
layout: post
title : 'Apache Parquet for GIS Data'
---

I recently graduated from the General Assembly Data Science Immersive Program. The program gave me a strong grounding in data science fundamentals and the skills and confidence I need to start independently researching areas of particular personal interest. One of which is Data Engineering. A contact of mine suggested that I learn about working with [Apache Parquet](https://parquet.apache.org/) files. I recently completed a project using [Ookla network performance data](https://github.com/teamookla/ookla-open-data).I had originally used [ArcGIS Shapefiles](https://desktop.arcgis.com/en/arcmap/10.3/manage-data/shapefiles/what-is-a-shapefile.htm), but the data was also available as Parquets. I decided to rerun the most memory intensive step of the project, loading the file into a GeoPandas dataframe, using parquets to see what impact it had on performance.

## The Dataset


The project considered global fixed broadband speeds for the first quarter of 2020. It consisted of 6,688,699 rows and 7 columns. Each columns represented a map tile and that tiles coordinates as a Quad Key, its boundaries as Latitude and Longitude, average upload speed, download speed and latency and the number of tests performed on that tile.

## The Code

 The initial code was very simple thanks to GeoPandas having good support for reading remote Shapefiles. All code is written in Python 3.7.

{% highlight python %}
import geopandas as gpd
import timeit
ook_url = "https://ookla-open-data.s3.amazonaws.com/shapefiles/performance/type=fixed/year=2020/quarter=1/2020-01-01_performance_fixed_tiles.zip"
start_time = timeit.default_timer()
q1_2020 = gpd.read_file(ook_url)
print(timeit.default_timer() - start_time)
{% endhighlight %}

Rewriting it to work with parquet files was slightly more complicated, because there doesn’t seem to be a simple way for GeoPandas to directly load parquets where the geometry is encoded in the wkt format. If you have a more efficient solution, please contact me and let me know.


{% highlight python %}
paqs = "https://ookla-open-data.s3.amazonaws.com/parquet/performance/type=fixed/year=2020/quarter=1/2020-01-01_performance_fixed_tiles.parquet"
start_time = timeit.default_timer()
q1_paq = pd.read_parquet(paqs)
from shapely import wkt
q1_paq['tile'] = gpd.GeoSeries.from_wkt(q1_paq['tile'])
q1_paq_geo = gpd.GeoDataFrame(q1_paq, geometry = 'tile')
print(timeit.default_timer() - start_time)
{% endhighlight %}


## The Results

Even with the somewhat more complicated code, the parquet performed much better than the shapefile. It took about 411 seconds to create the dataframe from the shapefile, and about 224 seconds to do the same with the parquet. This shows that parquets can substantially reduce overhead. Using parquets will substantially improve my ability to work with medium to large datasets.