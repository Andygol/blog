---
layout: "post"
title: "OpenStreetMap ain't Google Maps"
date: "2023-07-22 10:30:00 +0200"
categories: en
tags:
  - OpenStreetMap
  - Google Maps
  - OSM
  - API
comments: true
lang: en
ref: osm-not-google-maps
---
> *There should be a screenshot from Toy Story where Buzz tries to explain to Woody something about the stars by showing his hand up.*

About 20+ years ago, I came across a short fiction story on a computer-related theme with elements of a thriller, in which there was a moment when the main character hastily created some geoservices, applications that use them, almost from nothing, to get out of situations in which he found himself. At that time, I did not fully understand the whole idea, because I had never encountered such a thing in my life. And here we are in the new century/millennium, we have a device in our pocket that uses these mysterious geoservices — a smartphone.

Let's try to figure out these mysterious geoservices. In fact, they existed long before the appearance of smartphones, and even before the appearance of computers in general. The simplest example is postal addresses. Knowing the number of the house, the name of the street, you can get to the destination. You can plan (think over) the route on your own by drawing an approximate diagram on a piece of paper, or by making a similar one in your head. In fact, we do this every time we leave the house – we plan our route and then stick to that plan, even when we go to the nearest store to buy groceries. If you are going to an unfamiliar, remote place about which you either have little information, or you have never been there, you ask your friends how to get there, what type of transport to use, which stop to get off at, which direction to go next and where to turn. You can take a taxi and the driver will take you to the specified address or landmarks. If you need to send a parcel to another city, the postal service will provide you with such a service, but you still need to specify the recipient and the place where the parcel should be delivered. As you can see, we need information about the location of some object in the area (in space), that is, geospatial information. This information is the basis of all geoservices.

> … then there will be a beet field, then a forest strip, then a corn field, then a wheat field, a hemp field, behind it a talking river — it will tell …

Geoservices are information and technological solutions that provide collection, processing, storage and provision of geographic (geospatial) information. They help to understand and use spatial data for various purposes such as navigation, data analysis, planning, marketing and many more. With the help of geoservices, users can interact with maps, exact coordinates, geographic objects and other geodata.

And now it is almost impossible to imagine calling a taxi, ordering a pizza delivery, and calculating the potential reach of the target audience without the use of geospatial data and services.

I have repeatedly seen people confuse data with services. This is exactly the case mentioned in the title. I have seen several different analyses where the authors tried to compare Google Maps exactly with OpenStreetMap without having any idea of what they were comparing. Let's figure it out.

## Let's talk about Google Maps first

The history of Google Maps began in the mid-2000s and is an interesting example of a successful combination of innovative technology and practical application.

Google Maps was launched on February 8, 2005. The maps were created as a project of the Google team, which sought to create a map web service with high map loading speed and interactive capabilities.

The first version of Google Maps provided the ability to view maps, search for places, plan routes, and navigate using GPS. The interface was as intuitive and user-friendly as possible.

As early as June 2005, Google introduced the Google Maps API, which allowed developers to integrate mapping functions into their websites and applications. This opened wide opportunities for the use of cartography in other projects.

In 2007, Google Maps was supplemented with a new functionality — Street View, which allowed users to view photos along roads and streets, which gave them an idea of ​​a realistic view of the area, there was an opportunity to virtually visit other places in which you have never been.

In 2008, a mobile version of Google Maps for smartphones was launched, allowing users to access maps and navigation directly on their mobile devices.

In 2010, Google integrated the functionality of Google Maps with another product — Google Earth, allowing users to view 3D models and images from satellites.

Google Maps continued to evolve, adding new features such as turn-by-turn navigation, offline mode, place ratings and reviews, event and establishment recommendations, and more.

Today, from the point of view of the average user, Google Maps has become one of the most popular tools for navigating and exploring the world. They are used by millions of people every day to find directions, find places, view photos and explore new remote locations. Google Maps has also become an important resource for businesses, researchers and public organizations that use its services to solve various tasks.

## OpenStreetMap

The history of OpenStreetMap (OSM) begins in 2004 and is connected with the desire to create a free, open and accessible database of geospatial information.

OpenStreetMap was founded in Great Britain by Steve Coast in August 2004. He had the idea to create a global mapping platform where users could collaboratively create and edit geodata.

The main reason for the emergence of OpenStreetMap was the lack of available and up-to-date geographic information in some regions of the world. Commercial global mapping services such as Google Maps (which did not exist at the time) did not always provide coverage of sufficient quality or detail for certain areas.

OpenStreetMap put forward the principle of open data and access to cartographic information. The project became a platform for a global community of volunteers who could join the process of creating and editing geodata of any region of the planet.

Over time, OpenStreetMap attracted the attention of more and more users and built an active community. Volunteers began to actively enter data on roads, localities, information on hydrography, enterprises, institutions and other geographical objects.

OpenStreetMap's development-oriented community has become one of the largest and most active free mapping communities in the world. Through open and free data access, openness and collective efforts, OSM has become a source of valuable and relevant geospatial information used in a variety of industries, including tourism, research, humanitarian aid and urban planning.

## Goals and philosophy

Google Maps is a commercial product created by Google. The main goal of Google Maps is to provide users with convenient and fast interactive maps for navigation, finding places and orientation. Google makes money from it by displaying ads and providing paid services.

OpenStreetMap, on the other hand, is a project based on volunteer principles and an open data philosophy. The main goal of OSM is to create a free and publicly available geospatial database for the entire world. Everyone can contribute by adding new data or correcting issues. This approach makes it possible to create detailed and up-to-date maps where other sources may be limited or outdated.

## Data sources

Google Maps uses commercial and proprietary data sources, licensed data from third-party companies, and machine learning algorithms to process large volumes of data accumulated by the company along with on-site data collection using specialized equipment and customer device tracking. This gives them an advantage in speed and coverage, but hides details and sources of information. (Thus, all of us moving around the area unknowingly help Google sell us services using data received from our smartphones in the form of automated reports from applications or the OS.)

OpenStreetMap uses open data sources, such as satellite images provided by suppliers for the purposes of the project, open geospatial data distributed freely by national governments, data from other free open sources, as well as own contributions of project participants. This allows for more transparent and verifiable data that can be useful to the public, researchers and even humanitarian organizations.

## Data relevance

Google Maps has access to resources that allow updating their database with a certain periodicity, especially where there is an increased demand for them and more money can be made from the services. This helps provide users with up-to-date information such as traffic jams, entertainment and new establishments. (Remember your smartphones that Google collects data on all of this?)

OpenStreetMap, depending on the region, may have a different level of data relevance. Here everything depends on the number and activity of volunteers who make changes to the database. However, where there is an active community of project participants, the data can be quite relevant and detailed.

You can notice that both Google Maps and OpenStreetMap have similar elements on the main page:

- Map
- Search
- Ability to build a route
- View information about the object on the map…

## So what's the difference then?

The main difference is that Google Maps is a commercial product whose purpose is to provide services to customers. OpenStreetMap is primarily focused on data collection and distribution.

> – And what about the API? - you can say.  
> – This is not a single API, but a collection of various APIs, each of which is responsible for providing one or another service.

API for displaying maps; API for search (geocoding); API for routing; APIs for routing and providing turn-by-turn instructions and everything else are all these components that make up the final product.

Google's advantage is centralization, all APIs are developed and maintained in-house. Using them, you expect to receive a certain volume of services with a certain level of availability and quality (SLA), for which you are ready to pay a certain amount, without the need to deploy the entire infrastructure at your own or at the client's. Calculating how much it will cost you to use the services is quite non-trivial, because sometimes it is quite difficult to understand how some services depend on others and to predict the expected level of costs.

Instead, the OpenStreetMap API is intended for receiving, editing and saving data in a common geospatial database by project participants. That is, the only thing that OpenStreetMap guarantees you is that you can use the data and already using this data you can create your own map, geocoder, offer your customers navigation services or something else. The map, search, routing on the Main page of the OpenStreetMap site is a demonstration of what can be done with project data, all these elements are separate independent projects in themselves and the only thing that connects them to OpenStreetMap is the data that OpenStreetMap provides to everyone.

- Map — tile server ([Mapnik](https://mapnik.org/)) and map style ([OSM-Carto](https://wiki.openstreetmap.org/wiki/Uk:Standard_layer)) — 2 separate projects, which are in no way dependent on OSMF (OpenStreetMap Foundation)
- Tile Display Library — [leaflet.js](https://leafletjs.com), an open source JavaScript library for displaying interactive maps
- Search — geocoder [Nominatim](https://nominatim.org/), also an independent project
- Routing — [OSRM](https://project-osrm.org/), [Valhalla](https://valhalla.github.io/valhalla/), [GraphHopper](https://www.graphhopper.com /), also third-party projects
- Data export/mining — [Overpass API](http://overpass-api.de/) ([Overpass-Turbo](https://overpass-turbo.eu/)), ditto…
- [iD](https://ideditor.com/) is an OSM data editor (built into Main), now managed by OSMF.

All these are free projects with open source code and you, if necessary, if possible, if you have the appropriate qualifications, you can assemble the solution you need from these building blocks. Especially since you have a choice among various data rendering projects, map creation; various geocoding and search engines; routing and navigation engines. In addition, you can use OpenStreetMap data for analysis in your own projects, combining it with your other data, which is something you cannot do with Google Maps.

## Using OpenStreetMap

OpenStreetMap is used by various companies and organizations, including Meta, Apple, Amazon, TomTom and others.

Meta uses OpenStreetMap data for its mapping and location integration features. To provide its users with up-to-date information about places and locations; Meta uses OSM data to show users' locations, maps and integrate with other features on its platforms.

Apple also uses OpenStreetMap data in some of its services. For example, in some countries or regions where data from third-party providers for Apple Maps may be less complete or out of date, or conversely where OpenStreetMap data is of higher quality, they may use it to support navigation and cartography.

Amazon also uses OpenStreetMap data in its services such as AWS (Amazon Web Services). OSM can be used in a variety of geographic data processing, analysis, and visualization solutions on the AWS platform.

Among other manufacturers of navigation equipment, TomTom also cooperates with OpenStreetMap. They use OSM data to provide navigation and mapping services in their devices and applications.

Airbnb, a popular accommodation booking platform, also uses OSM data to display locations and accommodations on its maps. They can use OSM to provide accurate information about the location of residences and their surroundings.

ESRI ArcGIS Online allows users to integrate OpenStreetMap data into their geographic information systems. Users can import OSM data into their projects and use it for analysis, mapping, and visualization.

These examples show a wide range of uses of OpenStreetMap by well-known companies and services. The openness and availability of OSM data allow different platforms and organizations to use geographic information to provide quality services and develop their own products.

It should be noted that the choice of what to use, Google services, or OpenStreetMap data and ecosystem, depends on specific requirements, the completeness and relevance of data coverage in a particular area, the need for quickly received changes in data, the possibility of implementing data quality control processes and availability you have experience working with Google products or the OpenStreetMap ecosystem and data. The choice is yours.
