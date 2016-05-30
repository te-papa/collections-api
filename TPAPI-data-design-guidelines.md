# Te Papa API data design guidelines

#### Version 0.1

#### Revision History

Version | Date | Notes
--- | --- | ---
0.1 | 2016-05-30 | Initial draft

#### Author

Douglas Campbell

#### Licence

These guidelines are licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

## Abstract

A set of selected best practices in API data design to improve an API's usability (and uptake) by API consumers. 

## Guidelines

### 1. Introduction

A common goal for exposing your API is to gain wider exposure for your content. 
This means you are relying on 3rd party developers to use your API so that you reach your goal of increased exposure.
You can't force developers to use your API, but you can reduce any barriers and make it easier for them to use.

We are already familiar with designing the 'user experience' for visitors using a website, so we can also design the user experience for developers consuming an API.
The same principles apply, e.g. digestability, clarity, trust, familiarity, and delight [[UX-PRINCIPLES](#UX-PRINCIPLES)].

This document aims to define some best practices for the lower-level design of endpoints, data structures, etc. to increase the usability for API consumers.
There are many guidelines covering high-level API design (e.g. REST) &ndash; see [Related high-level API guidelines](#related-guidelines) below.

The examples below are presented as JSON, but apply equally to other data formats.

### 2. Attribute names

### 2.1 camelCase

Attribute names are formatted in 'camelCase'.

*Reason:* APIs are often consumed using Javascript, where camelCase is a common variable naming format.

```json
{ "familyName": "McCahon" }
```

*Not:* ~~family_name~~

### 2.2 Self-describing attributes

Attribute names use full, descriptive words.

*Reason:* Intuitive attribute names makes development easier as the API consumer doesn't need to constantly refer to the documentation to understand what they are.
It may increase the response size, but not by much.

```json
{ "philatelyGibbonsNumber": "R2898-11" } 
```

*Not:* ~~phlGb~~

### 2.3 Core attributes

Every object describing a thing contains the following core attributes: `id, type, title, href`

*Reason:* It is easier for an API consumer to 'hack' an initial version when they don't need to worry about which attributes are available for each object type.
Duplication of data isn't a big issue.

```json
{
  "id": "1234",
  "type": "Person",
  "edmType": "Person",
  "title": "Colin McCahon",
  "name": "Colin McCahon",
  "href": "https://collections.tepapa.govt.nz/agent/1234"
}
```

### 3. Data formats

### 3.1 Multiple formats

Provide the data formats that your API consumers need.

*Reason:* JSON is commonly used in APIs and may suit web developers, but it disadvantages API consumers using tools that only handle another format, e.g. XML. 

### 3.2 Dates

Structured date values are formatted as one of: `xsd:date, xsd:time, or xsd:dateTime`. See [[XML-SCHEMA-DATATYPES](#XML-SCHEMA-DATATYPES)] 

Other dates/times fragments are formatted using the W3C Date Time Format (a profile of ISO 8601) preferably in UTC. See [[W3C-DATETIME-FORMAT](#W3C-DATETIME-FORMAT)] 

*Reason:* The XML Schema and W3C date formats conform to ISO 8601, giving good interoperability with many programming languages and databases.  

```json
{
  "birthDate": "1987-07-16",
  "dailyAlarmTime": "19:20:30.85Z",
  "modified": "1987-07-16T19:20:30.85Z"
}
```
  
### 4. Structure

### 4.1 Resource endpoints

The number of endpoint IRIs (URLs) is kept to a mininum &ndash; new endpoints are only created if it is a fundamentally different resource.

*Reason:* It takes more programming code to query multiple endpoints and then merge the results back together, it's easier (and faster) to add filters to a single resource endpoint.

```
/items
```

*Not:* ~~/artifacts and /specimens~~

### 4.2 Embedded objects

The object structure is kept as flat as possible &ndash; grouping attributes into embedded objects is used sparingly.

The thing's attributes are presented as immediate child attributes. It is possible to visually group them by using similar attribute name prefixes.
However, related things that exist separately IRL (in real life) may make more sense as an embedded object.

*Reason:* More embedding requires more programming code to navigate through the levels of objects. It's easier when most attributes can be access directly from the object root.

```json
{
  "artworkMedium": "Acrylic",
  "artworkSupport": "Plywood",
  "licence": {
    "name": "Attribution 4.0 International",
    "uri": "https://creativecommons.org/licenses/by/4.0"
  }
}
```
*Not:* ~~"artworkDescription": { "medium": "Acrylic", "support": "Plywood" }~~

### 4.3 Concrete typed sets

Sets (arrays) containing mixed types of things are split into separate, concretely-named subsets.

Arrays only contain things that are of the same type (not because they happen to be stored in the same database table).
The API consumer can always re-combine them if necessary for their application.

*Reason:* It takes more programming code to filter a set to only retrieve things of a particular type, it's easier to retrieve a particular type by attribute name then just iterate through the member set.

```json
{
  "artists": [
    {
      "id": "1234",
      "name": "Colin McCahon"
    },
    {
      "id": "5678",
      "name": "Rita Angus"
    }
  ],
  "publishers": [
    {
      "id": "3456",
      "name": "Te Papa Press"
    }
  ]
}
```

*Not:* ~~"agents": [ {"type":"artist","name":"Colin McCahon"}, {"type":"artist","name":"Rita Angus"}, {"type":"publisher","name":"Te Papa Press"} ]~~

### 4.4 Expanding objects

Provide options to give the API consumer control over expanding to include related objects.

*Reason:* Some API consumers only need one attribute from a resource, others need attributes from this resource as well as those in directly related resources.
Some API consumers use a fully flexible development environment, others are using restricted tools.
Compiling related resources into a single, expanded data object is a common task &ndash; sometimes API consumers want it done for them, other times they don't.

Example options:

#### a. Identifier only

```json
{
  "name": "Colin McCahon",
  "related": [
    {
      "id": "1234"
    }
  ]
}
```

This provides a fast response, but to present a list of related people requires multiple additional API calls to expand each.

* Good if you just want the resource's attributes
* Bad if it is difficult for you to make additional API calls.

#### b. Expanded

```json
{
  "name": "Colin McCahon",
  "related": [
    {
      "id": "1234",
      "name": "Anne Hamblett",
      "roles": [
        "artist"
      ]
    }
  ]
}
```

This is complete, but the response may be slower (due to compiling all the related resources on the server before sending the response).

* Good if it is difficult for you to make additional API calls
* Bad if you just want the resource's attributes (slower and larger response to include unnecessary data).

### 4.5 Headers

All API-relevant information in the HTTP response headers is repeated in the response body.

*Reason:* Some API consumers use tools that don't provide easy access to the response headers &ndash; they aren't disadvantaged if they can also access that information via the response body.

```json
{
  "response": {
    "httpStatusCode": 200,
    "xRateLimitLimit": 10,
    "links": [
      {
       "rel": "self",
       "href": "https://collections.tepapa.govt.nz/agent/1234"
      }
    ]
  },
  "id": "1234",
  "title": "Colin McCahon"
}
```

### 4.6 Metadata

Housekeeping attributes of the object itself are kept separate from the main data (i.e.attributes of the thing the record describes).

*Reason:* Self-describing data is easier to develop with than having to constantly refer to the documentation to understand its attributes.
It's easier when the object structure clearly separates out attributes of the thing vs. attributes of this data object.

```json
{
  "title": "Central Otago",
  "creator": "Rita Angus",
  "modified": "2000-01-01",
  "metadata": {
    "creator": "dave",
    "modified": "2016-04-25"
  }
}
```

In this example, the "Central Otago" painting by Rita Angus was modified on 1 January 2000, this record about that painting was created by dave and was modified on 25 April 2016.

### 4.7 Arrays

Arrays are only used when there may be more than one occurence. 

For example, if the source database contains multiple records but only the first record is exposed in the API, it is presented as an object rather than an array containing a single object.

*Reason:* Processing arrays requires more programming code, which is wasted effort if there will only ever be one value.

```json
{
  "creator": {
    "id": "1234",
    "name": "Colin McCahon"
  }
}
```

*Not:* ~~"creator": [ {"id":"1234","name":"Colin McCahon"} ]~~

## <a name="related-guidelines">A. Related high-level API guidelines</a>

* [18F API Standards](https://github.com/18F/api-standards)
* [White House Web API Standards](https://github.com/WhiteHouse/api-standards)
* [Web API Design ebook](http://apigee.com/about/resources/ebooks/web-api-design)
* [JSON API specification](http://jsonapi.org/format/)
* [Atlassian REST API Design Guidelines version 1](https://developer.atlassian.com/docs/atlassian-platform-common-components/rest-api-development/atlassian-rest-api-design-guidelines-version-1)
                        
## B. References

**<a name="UX-PRINCIPLES">[UX-PRINCIPLES]</a>** 
[5 Simple UX Principles to Guide your Product Design](https://www.sitepoint.com/5-simple-ux-principles-guide-product-design/)

**<a name="XML-SCHEMA-DATATYPES">[XML-SCHEMA-DATATYPES]</a>** 
[XML Schema Built-in datatypes](https://www.w3.org/TR/xmlschema-2/#built-in-datatypes)

**<a name="W3C-DATETIME-FORMAT">[W3C-DATETIME-FORMAT]</a>**
[W3C Date and Time Formats](https://www.w3.org/TR/NOTE-datetime)
