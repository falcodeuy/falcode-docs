---
layout: default
title: Postgis (maps and locations)
parent: Integrations
grand_parent: Django
nav_order: 2
---

# PostGIS Integration

## Environment Setup

### Docker Configuration

Below are the changes needed to set up the environment for PostGIS and GDAL.

#### Dockerfile Modifications

Add the necessary dependencies for GDAL:

```Dockerfile
# Install GDAL and related dependencies
RUN apt-get update && apt-get install -y binutils libproj-dev gdal-bin libgdal-dev

# Set environment variables for GDAL
ENV GDAL_LIBRARY_PATH=/usr/lib/libgdal.so
ENV GEOS_LIBRARY_PATH=/usr/lib/libgeos_c.so
```

#### docker-compose.yml Modifications

Replace the default database image with the official PostGIS image:

```yaml
services:
  db:
    image: postgis/postgis:latest
    restart: always
    ports:
      - "5433:5432"
    env_file:
      - .env
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

## PostGIS Integration in Django

### Step 1: Use PointField and PolygonField in Models and Serializers

Below is an example of how to define models in Django that include geospatial fields such as `PointField` and `PolygonField`.

#### Models

```python
from django.contrib.gis.db import models

class Location(models.Model):
    name = models.CharField(max_length=100)
    position = models.PointField()  # Field to store a geospatial point

class Region(models.Model):
    name = models.CharField(max_length=100)
    area = models.PolygonField()  # Field to store a geospatial polygon
```

#### Serializers and forms

If you are using serializers you can integrate this easily with

```python
from rest_framework_gis.serializers import GeoFeatureModelSerializer
from .models import Location, Region

class LocationSerializer(GeoFeatureModelSerializer):
    class Meta:
        model = Location
        fields = ('id', 'name', 'position')
        geo_field = 'position'

class RegionSerializer(GeoFeatureModelSerializer):
    class Meta:
        model = Region
        fields = ('id', 'name', 'area')
        geo_field = 'area'
```

And they also provide the form alternative

```python
from django import forms
from django.contrib.gis import forms as gis_forms
from .models import Location, Region

class LocationForm(forms.ModelForm):
    position = gis_forms.PointField(widget=gis_forms.OSMWidget(attrs={
        'map_width': 800,
        'map_height': 500,
    }))

    class Meta:
        model = Location
        fields = ['name', 'position']

class RegionForm(forms.ModelForm):
    area = gis_forms.PolygonField(widget=gis_forms.OSMWidget(attrs={
        'map_width': 800,
        'map_height': 500,
    }))

    class Meta:
        model = Region
        fields = ['name', 'area']
```

### Example Requests to Create a Point and a Polygon

#### Create a Location (Point)

```python
POST /api/locations/
Content-Type: application/json

{
  "name": "Central Park",
  "position": {
    "type": "Point",
    "coordinates": [-56.18816, -34.9011]
  }
}
```

#### Create a Region (Polygon)

```python
POST /api/regions/
Content-Type: application/json

{
  "name": "Restricted Zone",
  "area": {
    "type": "Polygon",
    "coordinates": [[
      [-56.18816, -34.9011],
      [-56.19000, -34.9000],
      [-56.18900, -34.9020],
      [-56.18816, -34.9011]
    ]]
  }
}
```

### Example Geospatial Queries

Find the K Nearest Points

```python
from django.contrib.gis.db.models.functions import Distance
from django.contrib.gis.geos import Point
from .models import Location

# Define the reference point
reference = Point(-56.18816, -34.9011, srid=4326)

# Find the 5 nearest points
nearest_locations = Location.objects.annotate(
    distance=Distance('position', reference)
).order_by('distance')[:5]
```

#### Find Points within a Certain Distance

```python
from django.contrib.gis.measure import D

# Find locations within 1 km of the reference point
nearby_locations = Location.objects.filter(
    position__distance_lte=(reference, D(km=1))
)
```

#### Check if a Point is Within a Polygon

```python
from .models import Region

# Check if the point is within any region
point = Point(-56.18816, -34.9011, srid=4326)
regions = Region.objects.filter(area__contains=point)
```
