# Python Editing

In case the geojson data needs to be edited, particularly in batches, geopandas would be convenient to visualize and manipulating the data. An example as follows:

```python
import geopandas as gpd

us_gdf = gpd.read_file("https://raw.githubusercontent.com/PublicaMundi/MappingAPI/master/data/geojson/us-states.json")
```

In order to show the US state population on the US map, I downloaded the US state population data from Wikipedia and read it in as a pandas dataframe.

```
$$ us_state_npop_df.head()

state	july2019_population
California	39512223
Texas	28995881
Florida	21477737
New York	19453561
Illinois	12671821
```
Merge the state population dataframe onto the geojson shape dataframe:
```python
geo_tooltop = us_gdf.merge(us_state_npop_df, how='left', left_on='name', right_on='state')#.drop(['name'], axis=1)
```

## Mapping with folium



```python
import folium
import branca.colormap as cmp
from folium.plugins import MarkerCluster

m = folium.Map(localtion=[48, -102], zoom_start=3, tiles='OpenStreetMap')


#myscale = (us_npop_nreg['n_regs'].quantile((0,0.1,0.25, 0.5, 0.75, 0.9, 1.))).tolist()


marker_cluster = MarkerCluster(name="State Population").add_to(m)
for row in example_data.itertuples():
    folium.Marker(location=[row.LAT, row.LNG], popup="State Population: "+str(row.us_npop)).add_to(marker_cluster)

folium.Choropleth(
    geo_data = us_gdf,
    name='Absolute Volume Choropleth',
    data = test_data,
    columns=['state','n_pop'],
    key_on='feature.properties.name',
    fill_color='BuPu', #YlGn #
    #threshold_scale=myscale,
    fill_opacity=0.5,
    line_opacity=0.2,
    nan_fill_color='#006994',
    legend_name = "Example Data Pop Mapping",
).add_to(m)

folium.GeoJson(example_data_df[['geometry','state','n_pop','npop_per_10k','male_ratio','median_age','race_white_ratio']],
               name='US Population Demographics',
               style_function = lambda x:{'weight':2, 'color':'#000000', 'fillOpacity':0.2},
               highlight_function = lambda x:{'weight':3, 'color':'#006994'},
              smooth_factor=2.0,
              tooltip=folium.features.GeoJsonTooltip(fields=['state','n_pop','npop_per_10k','male_ratio','median_age','race_white_ratio'],
                                             aliases=['State','# Population','# Test /10k Population','Male Ratio','Median Age','White Race Ratio'],
                                             labels=True,
                                             sticky=True,
                                             toLocaleString=True)
              ).add_to(m)

folium.LayerControl().add_to(m)
```
