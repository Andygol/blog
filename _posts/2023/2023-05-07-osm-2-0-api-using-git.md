---
layout: post
title: 'OSM 2.0 API using git'
date: '2023-05-07 10:29:00 +0200'
categories:
  - en
tags:
  - OSM
  - API
  - git
  - yaml
  - yml
comments: true
lang: en
ref: osm-api-git
---
The OSM 2.0 API (OpenStreetMap) is an application programming interface that provides access to geospatial data and its change history stored with git. The basic data structure in OSM should be based on dividing the globe into tiles, each of which is a separate git repository.

The OSM object data storage format is yaml (YAML Ain't Markup Language). Yaml is an easy-to-read and easy-to-save data format based on the use of indentation to represent data structure. Using yaml in OSM allows storing various multi-level attributes in the form of tags for geographic objects.

Each object will be stored as a separate YAML file, unlike previous versions of the OSM API where data was stored and exchanged as XML files.

Each tile in OSM is an independent git repository. Git is a version control system that allows you to track the history of changes in a file system and store different versions of files. Using git for OSM allows you to track all changes made to geospatial data and store their history.

## Description

The OSM API should provide various methods for accessing and modifying data. Some of these should include:

1. Obtaining data by coordinates or area.
2. Search for objects by type, attributes or tags.
3. Obtaining the history of changes for a specific object.
4. Addition of new objects or changes to existing ones.
5. Deleting objects.

The OSM API should provide the ability to perform queries based on various protocols to conveniently retrieve and interact with geospatial data.

Overall, the OSM API, which uses git to store geospatial data and its change history, provides a powerful mechanism for collaborating and managing geospatial data. Key features of the OSM API using git include:

**Version control**:
By using git, the OSM API allows you to store a change history for each object. This means that you can track who made changes to a specific object, when, and what changes, as well as restore previous versions of the data.

**Distributed data structure:**
The globe is broken down into tiles, which are self-contained git repositories. This allows you to effectively manage and distribute the load on different servers. Each tile contains only the objects within its boundaries, which makes it easier to access specific data.

**Data format flexibility:**
Using the yaml format to store object data allows you to store various attributes and tags that help describe geographic objects in detail. This allows you to add your own tags and attributes for more flexible data presentation.

**Scalability:**
The distributed data structure and the use of git make it easy to scale the system to handle large volumes of geospatial data. You can add new tiles or servers to expand the infrastructure and improve performance, break large tiles into smaller ones over time.

**Convenient access to data:**
The OSM API provides a variety of methods for accessing OSM data. You can retrieve data by coordinates or regions, search for objects by type, attributes, or OSM tags, and retrieve change history for a specific object. This provides flexibility and the ability to select only the data you need.

**Possibility of joint work:**
Using git allows multiple users to work with data simultaneously and make changes. Thanks to version control and the ability to merge branches, you can easily work together on projects, make and merge changes.

**Integration with other tools:**
Because OSM uses git, you can use a wide variety of tools that support git to work with geospatial data. These can be various version control systems, code editors, data analysis tools, and others.

In addition to these capabilities, the OSM API using git allows you to:

**Data extension:**
You can add your own data to OSM data, extending the current geospatial database. It allows you to create your own datasets, add attributes, and extend the functionality of OSM.

**Integration with other services:**
The OSM API can be easily integrated with other services and tools, such as geospatial services or data analysis systems. This allows you to combine different data sources and use them to create the right solutions.

**Advanced editing options:**
The OSM API allows you to make changes to geospatial data, including adding new features, editing attributes, and deleting features. This makes it possible to actively cooperate with the global OSM database and supplement it with knowledge about specific objects or regions.

These are just a few additional features of the OSM API using git. Overall, the OSM API should be a powerful tool for working with geospatial data, providing version control, data format flexibility, and scalability.

## YAML is for saving data

OSM API version 2.0 uses the yaml format. Each object is stored as a separate YAML file, and this differs from previous versions of the OSM API, where data was stored as XML files. Using the YAML format to store individual object files has several advantages:

**Ease of reading and editing:**
The YAML format has a clear, easy-to-read syntax that makes it easy to manually edit or view data. Using indentation instead of XML syntax elements makes it easier to work with data and edit it

**Flexibility in data expressiveness:**
YAML allows you to store additional properties and information about objects that are necessary for a particular application. It can include simple values, lists, associative arrays, and nested data structures.

**Data search and filtering:**
Being individual files, objects can be easily found and filtered by various parameters. You can use YAML libraries or tools to search and manipulate data as needed.

**Change history support:**
Each object is presented in the form of its own file, which allows you to save the history of changes and track the development of data. Git, as a version control system, can be used to manage changes and branches, which facilitates effective collaboration and data version control.

**Data expansion and validation:**
You can use YAML schemas to validate and control the correctness of data. This ensures that the data conforms to the given rules and structure. Also, you can extend YAML schemas to define additional properties and constraints for stored objects.

**Integration with other tools:**
YAML is a popular data exchange format supported by many tools and platforms. You can use various tools to import and export data, and to process and analyze geospatial data in YAML format.

![null island yaml](/images/2023/05/2023-05-07-osm-2-0-api-using-git.png)

## Git repository as a data store

The shape (geometry) of the tiles of each repository can be arbitrary.

In the OSM API using git, the shape (geometry) of the tiles of each repository can be arbitrary. Tiles can be of different shapes and sizes, depending on your geospatial data needs.

A description of tile boundaries, or other metadata that pertains to each tile, can be stored in the tile repository itself. This allows important information to be stored alongside geospatial data, helping to ensure consistency and availability of that data.

In addition, there is a master repository that includes tile repositories in the form of git submodules. The master repository serves as the starting point for managing and coordinating tile repositories. It may contain additional information about the tiles, such as indexes or links to each tile.

This architecture allows geospatial data to be organized and managed by storing the description of tile boundaries in the tile repository itself and using a master repository to coordinate and access these tile repositories.

This approach provides flexibility and scalability when working with geospatial data, allowing easy management of individual tiles, as well as maintaining change history and version control using git.

Data segmentation into tile repositories allows you to implement:

**Support for distributed cooperation:**
By using git, the OSM API enables distributed collaboration. Different users can work on different tiles, make changes and interact with the OSM database using the same principles as for working with code, and later merge their changes using git.

**Synchronize and merge changes:**
By using git, the OSM API makes it easy to synchronize and merge changes made by different users into their own copies of repositories. This avoids conflicts and ensures data integrity when merging changes. In addition, users can make their own forks to add their (mostly non-public) data on top of OSM data. Over time, when the decision is made to push your own closed data into the public domain, this can be done by using git push. Changes can be pushed both upstream, from local copies to shared repositories, and downstream, from shared to local repositories, allowing users to keep their own repositories up-to-date while receiving updates from the community, with less effort to synchronize data.

**History of changes:**
Thanks to the use of git, the OSM API stores the history of object changes in each tile, which allows you to follow the development and changes of geospatial data. This is a useful feature for analyzing data, tracking changes, and restoring previous versions.

**Version support:**
Git allows you to store each version of an object (file) in a separate commit, which makes it possible to easily review, restore, and track data changes over time. You can view the change history and go back to previous versions if needed.

**Easy replication and distribution:**
Because each object is stored in its own file, they can be easily replicated and distributed across servers or tiles. This allows for improved scalability and data access. Git provides the ability to synchronize and replicate data between different servers or tiles. You can use git features, such as branching and merging, to manage changes and updates to your data.

**Conflict management:**
In the case of simultaneous changes to the same object, git provides a means to resolve conflicting changes and merge different versions. This allows multiple users to work with the same data and handle conflicts efficiently.

**Data recovery:**
Thanks to git version control, the OSM API allows you to restore data to previous versions or restore deleted objects. This ensures data safety and security.

## OSM changeset – git commit

So, in the context of saving OSM data in git, a changeset is represented as a commit in git. A commit is a fixed data structure that captures the changes made at a specific point in time.

Each commit in git contains information about the changes made to the repository. In the case of OSM, a commit can contain changes related to the addition, deletion, or modification of geospatial objects.

Git commits provide a history of changes and allow you to track the development of data over time. Each commit has a unique identifier (hash), which serves to identify them and restore a certain version of the data.

One commit can contain changes for one or more geospatial objects. This allows you to record changes in individual objects and manage the history of changes independently of other objects.

Commits can be grouped into branches and merged with each other, which allows you to manage parallel branches and merge their changes into one common version of the data.

Hence, commits in git are used to save and manage changes to OSM geospatial data in the git version of data storage.

## Advantages of using git

**git commands:**
You can use standard git commands to manage tile repositories, such as `git clone`, `git pull`, `git push` and others. It allows you to work with the geospatial data of the OSM API using the familiar and powerful git interface.

**Advanced git features:**
The OSM API can use additional features and capabilities of git, such as branches, git-tags, and commits. It allows you to organize and manage different versions, create milestones and work with different branches.

**Access control:**
The OSM API can provide the ability to restrict access to individual tiles or data using git's access control features. This allows you to control who has the right to read, write or do other things with geospatial data.

**Backup and Restore:**
Thanks to git, the OSM API allows you to back up your data and restore it when needed. This provides an additional layer of protection and security for your geospatial data without having to develop your own solutions for it.

**Distribution of changes:**
You can easily propagate changes made to tile repositories to other OSM API instances or installations using familiar git methods. No need to waste server resources creating data dumps and data for replication. With the use of git, this is done by fetching updated data as needed with the git pull command. It allows collaboration with different communities or distributed systems, sharing and synchronizing geospatial data.

**Integration with other tools:**
The OSM API using git can be easily integrated with other development tools, such as project management systems, automation tools, cloud storage services, and others. This makes it an important component when developing geospatial applications and working with data.

**Synchronization with other data sources:**
Using the OSM and git APIs, you can synchronize geospatial data with other data sources. This can be useful if you have additional data sources or want to combine data from different sources to create a more complete and comprehensive database.

**Extensions and customizations:**
The OSM API with git allows you to extend and customize its functionality according to your own needs. You can add additional modules, functions, and extensions to work with geodata and interact with the API.

Using the OSM API with git provides powerful capabilities for managing geospatial data, maintaining its change history, and collaborating with the community.

### Summary of OSM API capabilities using git

The OSM API, which uses git to store geospatial data and change history, has the following features:

1. Each tile is an independent git repository containing a separate set of geospatial data.
2. Tiles can have a different shape (geometry) and the description of their boundaries is stored in the tile repository itself and the master repository.
3. Data about individual objects is stored in the YAML format, which allows you to store additional properties and information about them, imitating all the flexibility of the OSM tagging system.
4. The API supports distributed collaboration, allowing different users to work on different tiles and merge changes using git.
5. Changes are saved in the change history of git repositories, which allows you to follow the development of geospatial data.
6. Standard git commands like `clone`, `pull` and `push` can be used to manage tile repositories.
7. The API provides advanced git features such as branches, git tags, and commits for version control and development.
8. There is an option to restrict access to tiles and data using git access control.
9. The API allows you to restore data to previous versions and perform data backup and recovery.
10. Changes can be propagated to other instances of the OSM API and collaborated with the OpenStreetMap community and other developers.

## Distribution of large objects between tiles

The distribution of large objects between tiles is an important aspect when organizing geospatial data in the OSM API using git. This allows you to effectively manage large volumes of data and ensure optimal performance and speed of access to this data.

There are several approaches to distributing large objects between tiles in the OSM API. Here are some of them:

**Geographic distribution:**
Objects can be distributed between tiles using their geographic location. For example, large regional objects can be divided into separate tiles covering the corresponding territory. This allows data to be stored closer to its physical location and reduces the load on individual tiles.

**Distribution by categories:**
Objects can be distributed between tiles depending on their categories or types. For example, road or hydrographic features can be stored in separate tiles, making it easier to access specific categories of data.

**Dependence on size:**
Objects can be divided into tiles depending on their size or complexity. For example, large or complex objects can be stored in separate tiles for optimal management and quick access to them. It can be contours of oceans and continents, borders of countries and so on.

**Dynamic allocation:**
The OSM API can automatically allocate large objects between tiles based on load and demand.

An alternative approach may be to distribute large objects based on virtual tile boundaries. Instead of physically distributing objects between separate tile repositories, you can use virtual borders that join objects that are logically related to each other.

In this approach, large objects can be divided into multiple tiles, and these tiles can share a repository or have references to each other. This allows objects that are physically located in different tiles to be stored together in a virtual group.

Accordingly, you can have each tile as a separate git repository, but they can reference each other to form a virtual group of objects. This provides convenient access and management of large objects located in different tiles, while maintaining the flexibility and speed of git.

The choice of approach to the distribution of large objects between tiles depends on the specific needs and requirements for performance, availability, and manageability of the data.

## Notes

It is important to note that this description is general and specific details and implementation may depend on your specific OSM 2.0 API implementation and your needs.

When working with the OSM API using git, make sure you follow OSM's rules and policies for making changes to the global database. Keeping a change history and using git correctly will help manage these changes and ensure data security.

If you have specific questions about using the OSM API with git or need more details, please let me know and I'll try to give you more details.
