# Introduction #

The document outlines the goals and general code structure for the Google data client library. This document was part of the initial design process and the actual code in the client library may differ from this preliminary design.

# Table of Contents #
  * Project Objectives
    * Supported Properties
  * Overview
    * Sample Usage
  * Detailed Design
    * Classes

# Project Objectives #

Provide an open source library to assist Python developers in using Google data APIs. A secondary objective is to provide code for Atom and the Atom Publishing Protocol. This client library will form the foundation for the Google data Python sample code.

## Supported Properties ##

The following Google data web services are supported in this client library:
  * Google Base Data API
  * Google Calendar Data API

# Overview #

The client library contains two broad categories of classes: data models and service clients.

The service classes will provide methods to perform CRUD operations and queries on Google web services. They will also contain a class for carrying operations in the Atom Publishing Protocol.

The data model classes simplify the construction and manipulation of XML for the resources accessed through the web services. Data classes are provided for elements in Atom and Google data, as well as elements specific to services like Google Base and Google Calendar. Programmers may also modify the XML by accessing the ElementTree which the data classes wrap.

## Sample Usage ##

A sample's worth a thousand words, so here's an example of client code which adds an item to Google Base.

```
import gdata.base.service
import gdata.base
import gdata.service
import atom

base_client = gdata.base.service.GBaseService()
# ...
# Assign credentials, then login:
try:
  base_client.ProgrammaticLogin()
except gdata.service.CaptchaRequired:
# ...

# handle captch challenge if needed, then proceed to create a new base item programatically:
# (you could also build the item by parsing a string or reading a file)
new_item = gdata.base.GBaseItem()
new_item.author.append(atom.Author(name=atom.Name(text='Jeff Scudder')))
new_item.item_type = gbase.ItemType(text='products')
new_item.label.append(gbase.Label(text='Computer'))
new_item.link.append(atom.Link())
# ...

# Now add the item to Google Base
try:
  created_item = base_client.InsertItem(new_item)
  # delete the item we just created
  delete_succeeded = base_client.DeleteItem(created_item.id.text)
except gdata.RequestError:
  print 'Operation failed'

# Search for some items in the snippets feed.
base_query = gdata.base.service.BaseQuery()
base_query.feed = '/base/feeds/snippets'
base_query['max_results'] = 50
base_query['start_index'] = 100
base_query.bq = '[item type: digital camera]'
feed = base_client.Query(base_query.ToUri())
# Print the name of the first author in the first entry in the feed.
print feed.entry[0].author[0].name.text
```

# Detailed Design #

The client library is divided into the following modules:

  * atom: contains Feed, Entry, ... facilitates creation of Atom compliant XML objects
  * app\_service: contains AtomService which provides HTTP CRUD operations on Atom feeds
  * gdata: contains classes for Google data specific extensions to the Atom data model
  * gdata\_service: contains GDataService which is the foundation for the other service classes. Extends AtomService and adds Google authentication.
  * gbase: contains BaseItem, Label, etc.
  * gbase\_service: contains GBaseService and BaseQuery which supports base query syntax (?bq="...")
  * gcalendar: contains CalendarEvent, etc.
  * gcalendar\_service: contains CalendarService
  * blogger (not in initial release)
  * blogger\_service (not in initial release)
  * gspreadsheet: contains GSpreadsheetsSpreadsheet, GSpreadsheetsWorksheet, GSpreadsheetsCell, etc.
  * gspreadsheet\_service: contains GSpreadsheetsService

## Classes ##

### atom.Entry ###

A class representing an Atom Entry XML object. The Entry contains other Atom classes in the Atom module to represent the information in an Entry. It has class properties for all of the required and optional sub-elements (such as Id, Link, Author, etc.). An Entry can be instantiated by specifying elements like so:

`my_entry = atom.Entry(id=atom.Id(), title=atom.Title(text='My Entry'))`

Before being sent to the server in a POST or PUT, the Entry instance is converted to a string of XML, and the server's response (if the operation was successful) can be translated into an Atom Entry using the EntryFromString function.