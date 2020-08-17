---
title: Getting New York's road data into an iOS app
date: Last Modified
description:
    I'm teaching myself Swift and I inadvertently picked a weirdly-shaped
    project — a map-based app using as few 3rd-party libraries as possible.
    Getting New York's road data, block by block, turned out to be more
    elaborate than I thought it would be.

layout: layouts/article.njk
tags: post
---

<!--
#### TODO:
-   Figure out a footer
-   Commenting?
-   Set SEO data from article metadata
-->

# {{ title }}

I do a lot of work with web build systems and performance, but I got into
writing software because I like making products. With all the free time that the
pandemic has brought, I've been spending my off-hours teaching myself to write
Swift instead of screaming into the void. I missed writing product code and I
missed writing a compiled language with a nice IDE and a type system. I also
missed the speed and responsiveness that comes with native code that you only
get in the browser if you are extremely careful. So, I decided to try and build
an iOS app in [Swift][swift]. This post is the first of a handful where I try to
digest what I'm working on. Think of this more as a journal than any sort of
tutorial.

## What are you making then?

My goal is to make an app that will show me where I've been in New York on a
block-by-block basis. New York is big and I've only lived here for 5 years, so
an app that shows me where I've been might also show me what I'm missing.

{% figure "/swift/wow-a-prototype.jpg" "I could have picked a smaller phone." "A map of Brooklyn with a current location indicator. The roads around the indicator are highlighted in red." "mw6" %}

From a technical perspective, I want to try and use [SwiftUI][swiftui] to build
my app and [Core Data][coredata] to persist its data. I also want to avoid 3rd
party libraries if possible. If, like me, you're new to a lot of this, SwiftUI
is Apple's new view framework, and Core Data is Apple's built-in data
persistence layer. I won't go into a ton of detail on these libraries in this
post, but a little context seems important. I figure these constraints will
force me to get experience with the core pieces of iOS development, which is
most of why I'm writing this app in the first place.

## Getting New York's road data

To start, I need to think about how I want this app to work. Ideally, I want to
break streets up into segments by block. I'm defining a block here as **the
chunk of a street between any two adjacent intersections**. For example,
Division Avenue between Berry Street and Bedford Avenue is a block, as is Berry
Street between Division Avenue and S 11th Street. To do this, I'll need a way to
represent a road segment so that I can store metadata about it. I'll also need
to know every possible road segment in New York so that I know what I have and
have not visited. Getting at this data turns out to be not so straightforward.

### Apple Maps

I started by looking at what Apple would give me. When you want to render a map,
Apple provides you with a view. They also provide you with a way to search for
locations and a way to get directions between two locations.

However, there's no way to get at any of the data directly. If I want the names
of roads nearby, or even the name of the road that's closest to my location, I'm
out of luck. I needed to be able to get a catalog of road segments and identify
the closest one from wherever I am. This data takes a while to collect, and
anyone who's collected enough of it isn't willing to give it away, with the
except the beautiful people who maintain OpenStreetMap.

{% figure "/swift/osm-awesome.jpg" "Not the prettiest map tiles, but definitely the free-est." "a screenshot from openstreetmap.org centered on New York City" %}

### OpenStreetMap

[OpenStreetMap][osm] is like Wikipedia for street data. It seemed like a natural
place to look since its data is free to use and is constantly being updated. The
reality is that map data is really, really big, and getting at the data for just
New York City is gross. OpenStreetMap lets you [export][osm-export] regions of
map data, but exporting an area bigger than that a square mile or so ends up
being too big a request for them to process. My options were either to:

-   Download a lot of smaller regions and stitch them together.
-   Download [all of OpenStreetMap's data][planet] in one big ol' file and
    extract the data I needed.

Seeing as I didn't know much about how to use this data, neither option seemed
especially great. On the bright side, there are a few third parties that [will
extract data][bbbike] from OpenStreetMap for popular regions like cities. I
ended up finding and downloading some of the data for New York City, which ended
up being extremely detailed. The data included road segments, neighborhood
names, building shapes, and everything else you might imagine needs to be
represented on a map. It was also about a gig and a half of JSON, and while this
level of detail is great, I only really needed data on roads.

### New York's Centerline Data

Around this time, I learned about New York City's Open Data initiative. They
host a project called [The NYC Street Centerline][centerline], which holds data
about New York's roadbeds. It's a fantastic start, but it's not as polished as
OpenStreetMap's data. For example, take a look at this screenshot of NYC's data
on top of Google Maps. The blue lines are roads rendered from Centerline data.
Note the discrepancies, especially around driveways, walking paths, and parking
lots.

{% figure "/swift/nyc-centerline.jpg" "[waves hands] 'You get the idea, there's roads.'" "a screenshot from NYC Centerline showing a number of small roads with missing data" %}

It also appears to be missing a lot of data about roads through some parks and
cemeteries, which would be nice to know about. Here's what Woodlawn Cemetery
looks like, for example:

{% figure "/swift/centerline-parks-oh-no.jpg" "If you get pulled over in Woodlawn Cemetery speeding, you can tell the officer that the City of New York does not recognize these as roads and so it's impossible to violate road safety laws on them." "a screenshot NYC Centerline on a park in the Bronx. Road data is missing, but the map shows that there are in fact streets." %}

I'm betting OpenStreetMap's data is more accurate than NYC's here, but NYC's
data is a solid start. It's exclusively road data, so the actual file size is
about ten times smaller than the OpenStreetMap data for New York. Plus, it comes
with a bunch of fun attributes, like how wide the road is, the street numbers on
each side, and even what priority the street gets when snow needs to be plowed.
I'm betting I'll want to end up using OpenStreetMap at some point, but for now
this seems like a good way to get started.

## Making sense of a 134.5MB JSON file

Centerline hosts their data in a bunch of different types of formats, but most
of them looked like they'd require extra parsing to make sense of, as though
they were all just created from dumps of a [geospatial database][postgis]. For
example, JSON-formatted data was a list of un-keyed data, with the road's
coordinates encoded as a [Multilinestring][gis-multilinestring] that would need
to be parsed:

```json
[
    "row-ahmi-vsc6~mfe8",
    "00000000-0000-0000-058D-BBD264543C90",
    0,
    1597147596,
    null,
    1597147596,
    null,
    "{ }",
    "341",
    "391",
    "62180",
    "MULTILINESTRING ((-73.9210350736335 40.81165142603656, -73.92058708344143 40.81226350210869))",
    "354",
    "374",
    "10454",
    "10454",
    "1422604712",
    "1422603138",
    "WILLIS AV",
    "2",
    "1",
    "2",
    "64",
    1196294400,
    1586822400,
    "TW",
    "1",
    "13",
    "13",
    "C",
    null,
    null,
    null,
    "AVE",
    null,
    null,
    "WILLIS AVE",
    "WILLIS",
    "TW",
    "255.162909019"
]
```

However, they did offer their data hosted as [geoJson][geojson], which is a
specification for encoding geodata as json (surprise). It's a bit more verbose,
so the filesize is a bit bigger, but filesize is a bit less of a concern when
your data is already a hundred megabytes. In geoJson, that same road segment
above instead looks like this (sorry in advance for all the scrolling you are
about to do):

```json
{
    "type": "Feature",
    "properties": {
        "rw_type": "1",
        "rwjurisdic": null,
        "continuous": null,
        "created_by": "DCP",
        "l_low_hn": "393",
        "modified_b": "BlockfaceID_LOAD",
        "bphys_id": "0",
        "rsubsect": "1A",
        "collection": null,
        "twisted_pa": "N",
        "r_zip": "10454",
        "r_low_hn": "376",
        "bike_lane": "2",
        "nominaldir": null,
        "snow_prior": "P",
        "to_level_c": "13",
        "joinid": null,
        "r_blockfac": "1422605356",
        "seglocstat": null,
        "nonped": null,
        "b5_sc": "278150",
        "created_da": "2007-11-29T00:00:00.000Z",
        "fire_lane": null,
        "within_bnd": null,
        "shape_leng": "263.63804189756",
        "special_di": null,
        "streetwidt": "64",
        "streetwi_1": null,
        "b7_sc": "27815001",
        "segmentlen": "263.63813458",
        "stname_lab": "WILLIS AV",
        "modified_d": "2015-12-22T00:00:00.000Z",
        "status": "2",
        "r_high_hn": "410",
        "l_zip": "10454",
        "avgtravtim": "0",
        "lsubsect": "1A",
        "sandist_in": null,
        "fcc": null,
        "from_level": "13",
        "boroughcod": "2",
        "posted_spe": "0",
        "trafdir": "TW",
        "l_blockfac": "1422604337",
        "physicalid": "62181",
        "borough_in": null,
        "accessible": null,
        "truck_rout": null,
        "l_high_hn": "409"
    },
    "geometry": {
        "type": "LineString",
        "coordinates": [
            [-73.92058708344143, 40.81226350210869],
            [-73.92011935314727, 40.81289384215231]
        ]
    }
}
```

I had a lot of trouble figuring out how to open and explore this file. Getting
an understanding of its shape is a requirement if I wanted to parse it into any
sort of app, and to do that I needed to poke around. Opening the file in Xcode
caused it to crash pretty reliably. VSCode similarly hung for a while, and when
it finally responded a bunch of navigation features seemed to be disabled. To
help explore all of this data, I ended up using an app called
[Dadroit][dadroit], a JSON viewer specifically made for viewing large files
quickly.

{% figure "/swift/dadroit.jpg" "Once again: not pretty, definitely useful." "a screenshot of the Dadroit app showing a segment of road data." "mw6" %}

Using Dadroit gave me enough context on the shape of the data to be able to
munch through it in a simple Swift app. The parsing itself took a while, but I
got far enough into the data to be able to highlight a bunch of random roads
from it.

{% figure "/swift/random-roads.jpg" "This was so much harder than it looks." "A screenshot from the iOS simulator showing a map of New York with a bunch of random, sparse road segments highlighted." "mw6" %}

## What did we learn, Salem?

1. There's a lot of ways to represent geospatial data.
2. Some of those ways are better than others.
3. I'll probably have to use OpenStreetMap data at some point, but it's ok to
   take the easy way first.
4. Map data is both complex and precious.

I want to write about how I sped up parsing this data, and then how I queried it
for road segments close to where I am without using anything too fancy (like a
geospatially-aware database). At the same time, this post is kinda long and I
should go feed the dog or something. If you made it this far, thank you and let
me know if I did anything too surprising in the comments box below.

[swift]: https://developer.apple.com/swift/
[swiftui]: https://developer.apple.com/xcode/swiftui/
[coredata]: https://developer.apple.com/documentation/coredata
[osm]: https://www.openstreetmap.org/
[osm-export]:
    https://www.openstreetmap.org/export#map=13/40.7192/-73.9564&layers=C
[planet]: https://planet.openstreetmap.org/
[bbbike]: https://download.bbbike.org/osm/bbbike/NewYork/
[centerline]:
    https://data.cityofnewyork.us/City-Government/NYC-Street-Centerline-CSCL-/exjm-f27b
[gis-multilinestring]:
    http://help.arcgis.com/en/geodatabase/10.0/sdk/arcsde/concepts/geometry/shapes/types.htm
[postgis]: https://postgis.net/
[geojson]: https://geojson.org/
[dadroit]: https://viewer.dadroit.com/
