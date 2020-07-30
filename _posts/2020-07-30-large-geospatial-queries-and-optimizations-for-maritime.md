---
layout: post
title: "Serving complex geospatial queries in real-time"
date: 2020-07-29 20:49:08
image: 'geoports/cover.jpeg'
share_image: 'geoports/cover.jpeg'
description: "Different hammers for different nails, or a love letter to postgres"
tags:
- Postgres
- PostGIS
- Geospatial Queries
- GIS
- Databases
- Maritime
categories:
- Greywing
- Tales of Work and Woe
twitter_text:
---

## Beginnings - doing it yourself

One of the first things we built at Greywing that took it out of webserver-level resource use was the maritime routing tool. Given two endpoints, we used a modified version of [A\*](https://en.wikipedia.org/wiki/A*_search_algorithm) and a network of maritime routes and highways to route a vessel between those locations. This needed us to build our own database of over 100,000 ports and to map ocean routes at multiple levels of resolution, but it gave us something to build on top of. Over time it has expanded to include custom legs and additional customization of the route, but the output is still the same. A list of GeoJSON paths that describe the route of a vessel for each day of its voyage. Here's an example route that takes a vessel from Singapore (SGSIN) to Paris (FRPAR):

![Sample Route]({{site.url}}/assets/img/geoports/route.png)

The first purpose of this route was in finding incidents of piracy and hijacking along the vessel route. Simple searches were easy in the beginning, but as the database of incidents grew to over 5000 (or 9000), we built a better solution. Dividing the map into b-tree like segments and fencing them in to find matches enabled us to quickly eliminate points that were far outside the route of interest in real-time enough time bounds for our application (<250ms>). Here's an example of an incident:

![Incident]({{site.url}}/assets/img/geoports/incident.png)

Due to the complex nature of these routes, this was still a complex job - for us it involved taking each line segment and comparing it to points nearby to find the distance. Further slowdowns came when we switched from simple [Great-circle distance](https://en.wikipedia.org/wiki/Great-circle_distance#:~:text=The%20great%2Dcircle%20distance%20or,line%20through%20the%20sphere's%20interior) (points on a sphere) to accurate [Geographical Distance](https://en.wikipedia.org/wiki/Geographical_distance). Vincenty distance fails for antipodal points, but we can quickly eliminate these as not being of interest. [Karney's algorithm](https://link.springer.com/article/10.1007/s00190-012-0578-z) provides a better result, but was far too slow to perform large queries. We found that using great-circle to eliminate points before labelling with accurate distance worked best here. A point or two might fall through this filter, but the general patterns still held strong for route planning and risk estimation. As an example, here's what Day 1 looks like for the route above:

![Incidents]({{site.url}}/assets/img/geoports/incidents.png)

## Proximity Ports - standing on the shoulders of giants

I've covered the recent [difficulties with crew changes](https://hrishioa.github.io/one-million-mariners/) in a different post, but in tackling this problem we soon realised that we had the data needed to provide actionable insights on crew change ports, in a way that only we could. We had aggregated and cleaned structured data about past crew changes, port restrictions, visa restrictions and flight restrictions. Most of this information was available for the looking, but only a digital platform could query thousands of these points in multiple dimensions to find the right places. This was the next target, a system that would analyze the vessel route and return the most valid ports for a crew change, factoring in all known data about the vessel, ports and countries.

The first thing we did was to use the existing algorithm for piracy on analyzing ports. It would be an understatement to say it did poorly.

```bash
Route Length: 500 nm
Buffer Size: 10 nm
Port Query runtime: 20.5s
Ports Queried: 120000
Ports Selected: 250
```

Did I mention this is using great-circle distance? The next step was to stop just short of using full [GIS](https://en.wikipedia.org/wiki/Geographic_information_system) tools, and make use of the native [`earthdistance`](https://www.postgresql.org/docs/8.3/earthdistance.html) and `cube` extensions for Postgres. I'm still inexperienced with GIS, and the breadth of standards and additional code complexity posed a problem. We did not want to integrate something into the stack when we didn't understand it.

I will never stop championing [Postgres](https://www.postgresql.org/) as the ideal database, especially if you don't know what you're doing - even more so if you do not know what you will be doing. It's light and fast for most use cases, and it's a database you can fully grow into. Sure, it occassionally demands [child sacrifices](https://unix.stackexchange.com/questions/282155/what-is-the-out-of-memory-message-sacrifice-child), but that's the price to pay for a good relational data store.

![Give me your firstborn]({{site.url}}/assets/img/geoports/postgres_child.png)

Earthdistance made distance comparison easier and more optimized, even without specific indices for the queries being made. Things looked a little better, and with a reduction in complexity.

```bash
Route Length: 500 nm
Buffer Size: 10 nm
Port Query runtime: 5s
Ports Queried: 120000
Ports Selected: 248
```

Earthdistance uses great-circle distance, and it seems our implementation disagrees slightly. Running multiple queries this difference was minimal, but it was still too slow for our use case. The port query is simply to find the nearest ports to a route. Once this is done, they need to be evaluated for their suitability to a crew change.

However, this proved to be a major improvement to our waypoint matching functionality, where we matched individual waypoints to ports in order to provide port-related information to routes that stopped short.

![Port Matching]({{site.url}}/assets/img/geoports/portmatch.png)

Using waypoint matching, routes that go through unrecognized or unspecified locations can still be labelled with useful data, and using `earthdistance` reduced the runtime of this query by an order of magnitude (10ms to <1). Labelling became a lot simpler as well:

```sql
SELECT locode, lat, lng, point(12.45, 1.23) <@> point(lng, lat)::point AS distance FROM ports
```

However, these were point-to-point queries while proximity ports required us to buffer query complex paths to thousands of ports. In the end, we bit the bullet and took the time to understand (PostGIS)[https://postgis.net/]. Before I go into detail, I can tell you that PostGIS is a beast. Perhaps it's not complete by QGIS and ArcGIS standards, but it contained more than enough tools for our usecases. With support for GeoJSON and easy path queries, we first converted our paths to PostGIS' internal representation using [ST_GeomFromGeoJSON](https://postgis.net/docs/ST_GeomFromGeoJSON.html), then ran it agains the database of ports for matches within a certain range. The results exceeded expectations:

```bash
Route Length: 500 nm
Buffer Size: 10 nm
Port Query runtime: 0.01s
Ports Queried: 120000
Ports Selected: 240
```

Not to mention that the results were computed using accurate geographical distance and needed no additional correction. It seems that the queries are also heavily optimized - labelling the distances first and filtering on them takes over 2 seconds, while filtering first and labelling takes less than a millisecond. There is a write-up on how this was done, but it's now lost to coderot and the general insistence of the web on changing links. I was able to find it on The Internet Archive, and [it's well worth the read](https://web.archive.org/web/20131212151037/http://boundlessgeo.com/2012/07/making-geography-faster/).

Needless to say, this was integrated into our system and proximity ports were released a few weeks ago. You can now route a vessel and immediately see which ports have had past crew changes, which ones are open, and which ones are surprisingly closed but still conducting crewing operations:

![proxports]({{site.url}}/assets/img/geoports/proxports.png)

Green wings indicate open ports conducting crew changes, purple wings indicate open ports that have not conducted a crew change, and orange wings (when they appear) show closed ports that seem to be conducting crew changes. This feature has been in use in production for some time now, with no problems. However, when we tried to use it in our excitement in all the places the previous geo-search heuristic had been in operation, we found that it was slower for both the waypoint matching as well as the piracy indicator. In both cases, PostGIS was significantly slower - pointing to a sweet spot in database size where it became viable.

As an example, this is one of the results of our benchmark of waypoint matching:

```bash
Waypoint Matching with custom module: 9.1 ms
Waypoint Matching with earthdistance: 2 ms
Waypoint Matching with PostGIS      : 8.5 ms
```
The results varied with PostGIS sometimes outperforming our internal algorithm, but they were always beaten by earthdistance.

These are the results for piracy searches:

```bash
Incident search with custom module: 500 ms
Incident search with earthdistance: 5084 ms
Incident search with PostGIS      : 3000 ms
```

It could well be that we haven't optimized PostGIS to the fullest extent, and it's something we're working on. But it makes sense to us that the three problems are different in complexity and requirements, and as we get to larger and larger datasets we may find that one size fits all - just not for now.


(Something I will say about PostGIS, is that setting it up on our UNIX machines in the cloud was a breeze. No problems whatsoever. In stask contract, it took me a week to get it barely functional on my Mac. Brew refused to link my PostGIS installation with Postgres, insisting on installing *the same version* to a different folder but refusing to link that to the system. Compiling from source worked, but brew continued to remove PostGIS' feeble attempts to link itself to brew's Postgres, until I gave up and recompiled both outside of brew. There were enough problems along the way that were entirely El Capitan's fault, and hastened my exit from the crumbling Macbook ecosystem. More on that later.)