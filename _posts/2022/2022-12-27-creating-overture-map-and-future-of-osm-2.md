---
layout: post
title: What does Overture Map mean for the future of OpenStreetMap? (part 2)
date: '2022-12-27 16:00:00 +0200'
categories:
  - en
tags:
  - OSM
  - Overture Map
comments: true
lang: en
ref: osm-overture-map-2
---
# What does creating an Overture Map mean for the future of OpenStreetMap? (Part 2)

The Overture Map Foundation, about the creation of which [I wrote the other day](https://blog.andygol.co.ua/en/2022/12/24/creating-overture-map-and-future-of-osm/), should solve problems that are still not solved in OpenStreetMap. Hopes, reproaches from the members of the OSM community are only a reaction to what should have happened a long time ago. Perhaps the basis of such reactions was the mention of the future use of OSM data in Overture Map.

To be honest, Overture Map owes nothing to OpenStreetMap and vice versa. [Views from the OpenStreetMap Foundation on the launch of Overture](https://blog.openstreetmap.org/2022/12/22/views-from-the-openstreetmap-foundation-on-the-launch-of-overture/), which were published on the OSM Blog, are in the fairway of events, in which the main player is the Overture Map Foundation, now in the foreground is Overture Map and its founders.

## Why was there a request to create such a project at all?

I wrote that the Overture Map Foundation is a parallel project in relation to OpenStreetMap. But it is not quite so. If you look at the structure of the working groups, using terminology that is familiar to OpenStreetMap members, you will not see any group that seeks to replace or duplicate the functions of the OSMF working groups. Instead, working groups will be created to resolve the perennial problems that exist in OSM. These are problems with the data scheme, the approach to tagging certain objects on the map, combining disparate data with each other (here, not only and not necessarily combining with OSM data, but also combining data from project participants into a common data set), optimizing deployment and use of data by consumers is, by the way, about permanent identifiers, and about reducing overhead costs for data processing, checking data before their publication, all that, unfortunately, was not stated in OSMF as goals and development directions for all the years of the project's existence.

What I see when people far from OSM and online mapping start comparing OSM and, say Google Maps, is that they are trying to compare apples and pears, many things are not obvious to them. In order to compare two fractions, they must be reduced to a common denominator, but in this case there are no common denominators. Services are compared with data.

Yes OSM is not about services, it is about data. While, say, Google Maps is all about services.

When someone pays attention to OSM, they come with a simple request - "I need a map" on my site or in the application. At this moment, they still do not fully understand that by the time they get to the map, they will have to do a lot more. People ask for a map, and they're told here's the data, here's some libraries and instructions, take what you want. And you are standing on the bank of the river, you have a pile of boards at your feet, a box with nails, you should have taken a hammer with you, and the only thing you need is to cross to the other bank at the moment. At that time, a bright "Google Maps" ferry with a bottom overgrown with shells is standing nearby on the pier, which barely overcomes the obstacle, but the ticket for it is not free either. There are also several independent carriers with boats of varying degrees of leakiness. And among this is the consumer, who needs to make a choice, to whom to entrust his future, how to spend the limited funds he has now.

Let's look at the OSM site (<https://openstreetmap.org>)[^1], what is there from the OSMF?

- The map is the result of using the [Mapnik](https://mapnik.org) library, which is responsible for converting OSM data into a visual form. But this is only what is on the surface.
- Without a style, which is needed to tell Mapnik which data element to visualize and how, you can't do either - creating a style is a separate [project](https://github.com/gravitystorm/openstreetmap-carto).
- The [Leaflet.js](https://leafletjs.com) library is used to display tiles generated using Standard map style.
- Search - relies on a separate project to create an open geocoder - [Nominantim](https://nominatim.org).
- [Routing](https://wiki.openstreetmap.org/wiki/Routing) – services provided by third-party projects[^2][^3][^4] that use OSM data for this.
- The section where it would be possible to [get data](https://www.openstreetmap.org/export) leads either to third-party projects[^ot][^gf][^dl] or to a global [dump of the entire planet](https://planet.openstreetmap.org/), digest which you still need to know how.
- Section with [diaries](https://www.openstreetmap.org/diary) of users - which was supposed to be a social platform in the project, but never became.
- Data quality control and monitoring system - absent.
- The system for viewing the [history](https://www.openstreetmap.org/history) of changes is incomplete and not very useful for average users.
- A system for organizing joint mapping is absent.
- Built into the site [iD data editor](https://www.openstreetmap.org/edit) - now under OSMF control. The foundation hired an engineer to support it, but before that it was built on [a grant from the Khight Foundation](https://blog.mapbox.com/large-investment-in-openstreetmap-from-knight-foundation-cf7aa00534db) and there was a certain time without support.
- The site code itself is inseparable from the [API code](https://github.com/openstreetmap/openstreetmap-website). A limited circle of people is responsible for its maintenance, which is good, but they only have enough strength to maintain the infrastructure. I assume that they no longer have the energy or time for any innovations, and they are not obliged to do it either. They act on a voluntary basis without receiving any reward other than moral satisfaction and understanding of their own significance for the project.
  
We can call OSMF's contribution only the API, which is where it all started, which allows ordinary mappers to get a piece of data for editing and submit the results of their work to the project database, create data dumps.

You may ask, what does the Board of the OSMF manage then? By design, it should lead [working groups](https://wiki.osmfoundation.org/wiki/Working_Groups) to which significant powers are delegated, but because the Board has no leverage over the working groups, because everyone participates on a volunteer basis, it turns out that the Board concentrates its attention on the increasing number of participants of OSM, those who directly enter data into the general database and try to keep them in a more or less up-to-date state.

On the website of the [OSMF Foundation](https://wiki.osmfoundation.org/wiki/Mission_Statement) you can find the Mission Statement of the Foundation, the governing document, the provisions of which are followed by the Board.

>The OpenStreetMap Foundation is an international, not-for-profit, democratic organisation with the tasks of supporting the OSM project, running and protecting the OSM database, and making it available to all. The OSMF membership is open to all who want to support the project and participate in the OSMF’s democratic process.
>
>The OpenStreetMap Foundation is there to protect the OSM data to keep it Free and Open.
>
>The OpenStreetMap Foundation represents the OSMF members, and wants to grow the membership.

Such goals were appropriate at the initial stage, when the project was a joint initiative of like-minded people who decided to spend part of their free time on a common cause, when ideas were bubbling and boiling and there were not enough manpower and funds for their implementation. But with the project going beyond this circle, reaching the level where not only mapping and cartography geeks began to pay attention to it, but also government institutions, other volunteer organizations and initiatives, and big business, the vision, mission and goals of OSMF remained without changes. And now in order to stay in place you have to run twice as fast, and in order to improve the state of affairs - even faster.

## Let's go back to the Overture Map.

As I wrote above, [Overture Map](https://overturemaps.org) is not an attempt to usurp the authority of OSMF. The Overture Map Foundation, in my opinion, wants to solve those issues from which the OSMF distanced itself. The "gatekeepers" of the OSM [^gatekeepers] had previously demonstrated their hostility to the initiatives that the big players were trying to bring to the OSM. Augmenting data by infusing the results of automated satellite image processing, presented by Microsoft and Meta, has met with only negative reviews. Which is why Microsoft [published](https://www.microsoft.com/en-us/maps/building-footprints) the results of automated recognition of the buildings' footprints and [roads](https://blogs.bing.com/maps/2022-12/Bing-Maps-is-bringing-new-roads) in the form of open data sets and placed them on GitHub under the ODbL license, the license under which OSM data is licensed. Maybe someone from OSM will find the time and inspiration to transfer them to the project. Meta, due to a failed initial attempt to feed data directly into OSM and some pretty nasty feedback from the OSM community, is also publishing its [Daylight](https://daylightmap.org) dataset separately. Needless to say, after the [presentation](https://2022.stateofthemap.org/sessions/B7VADW/) in the summer at SoTM 2022 by Microsoft of the mapping platform [Map builder](https://www.bing.com/mapbuilder/), built using the software stack used by OSM, which provided for the transfer of contributions to OSM after they were checked for quality and absence of vandalism, in OSM [was blocked by Data Working Group](https://www.openstreetmap.org/user_blocks/5701). In this case, how should the companies that founded the Overture Map Fondation behave? If on the one hand they hear "please take our data, use it as you need, you can even make money on it", and on the other "we are quite picky about your participation in the project, because we (OSM community) do not want for you to gain control over us, you better give us what you have and we will continue to do what we know how to do."

Now OSM is just a bunch of data. Sometimes their quality is better, sometimes worse. The entire software stack, except for the API, is built outside of OSMF's influence and maintained by non-OSMF actors. Unfortunately, it seems that OSMF did not try to take steps that could improve the situation, improve project management.

It's time to change a T-shirt to a business suit. OSM is no longer a hobby project – it is a major player in today's world of geospatial data providers.

Overture Map under the auspices of the Linux Foundation can (should) become a showcase for free geospatial data and services, a place where you will be offered not only boards, nails and partial instructions on how to build a boat, but also services to transport you to the other shore, something similar to public transport compared to a private car. OSM, if nothing changes, will still supply boards and nails, metal - someone has to do it anyway, and Overture Map will be a ship's yard where you can order a ship for yourself, or handy tools to operate your own ocean liner, or it will be a bridge with which you can reach the opposite shore. Perhaps the Linux Foundation will become part of OSMF or vice versa, which will allow communication between data users and those who create them. I look with optimism to the future, to the future synergy between all those who change the world for the better.

----

[^1]: <https://wiki.openstreetmap.org/wiki/Browsing>

[^2]: <http://project-osrm.org>, powered by [FOSSGIS e.V](https://www.fossgis.de/) <https://routing.openstreetmap.de/about.html> 

[^3]: <https://github.com/valhalla/valhalla>, powered by [FOSSGIS e.V](https://www.fossgis.de/) <https://gis-ops.com/global-open-valhalla-server-online/>

[^4]: GraphHopper – <https://www.graphhopper.com/>

[^ot]: Overpass API – <https://overpass-api.de/api/map?bbox=-130.0,-60.0,130.0,60.0>

[^gf]: Geofabrik Downloads – <https://download.geofabrik.de/>

[^dl]: Other sources – <https://wiki.openstreetmap.org/wiki/Download>

[^gatekeepers]: See «OSM has Hidden Gatekeepers» з допису «Why OpenStreetMap is in Serious Trouble» від Serge Wroclawski, Fri 16 February 2018, <https://blog.emacsen.net/blog/2018/02/16/osm-is-in-trouble/>
