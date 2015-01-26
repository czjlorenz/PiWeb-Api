---
title: Handle Raw Data
---
##Endpoint information

Raw data as well as information about the raw data can be fetched, added, updated and deleted via the following endpoints. Filter can be set as described in the [URL-Parameter section](General-Information#raw-data).

URL Endpoint | GET | PUT | POST | DELETE
-------------|-----|-----|------|-------
rawData/*entity*/ *uuid*/*key* | Returns rawData file for the specified *entity* with the committed *uuid* and *key* | Updates the committed  rawData file for the *entity* with the *uuid* and *key* | Creates the committed rawData file(s) for the *entity* with the *uuid* | Deletes rawData file for the specified *entity* with the comitted *uuid* and *key* or all raw data if no *key* is given
rawData/*entity*/ *uuid*/*key*/ thumbnail | Returns a preview image of the raw data object for the specified *entity* with the committed *uuid* and *key* | *Not supported* | *Not supported* | *Not supported*
/rawData/*entity* | Returns a list of raw data information for the *entity* with the uuids commited within the url parameter section | *Not supported* | *Not supported* | *Not supported*

##Add raw data

Raw data can be added to all kinds of entities: part, characteristic, measurement and measured values.
On adding raw data further information need to be passed beneath the data itself. The information the raw data object belongs to is passed within the uri, the data are passed within the HTTP body and the additional information are passed within several HTTP header variables:

HTTP header variable | Description                                    | Example Value
---------------------|------------------------------------------------|--------------------------------------
Content-Disposition  | Includes the raw data object's file name       | "MetalPart.meshModel"
Content-Length       | Includes the raw data object's length in bytes | 2090682
Content-MD5          | Includes raw data object's MD5 hash sum        | "bdf6b06ab301a80ae55021085b820393"
Content-Type         | Includes raw data object's MIME type           | "application/x-zeiss-piweb-meshmodel"

On adding raw data it is possible to pass a key within the uri. If -1 or no key is passed the next free key will be used on server side. (recommended)

![Warning](images/warning.png) If a key is passed which is already in use the existing raw data object will be replaced.

### Add a raw data object to a part with the uuid b8f5d3fe-5bd5-406b-8053-67f647f09dc7

####Example for direct webservice call

Request:

{% highlight http %}
PUT /rawDataServiceRest/rawData/part/b8f5d3fe-5bd5-406b-8053-67f647f09dc7 HTTP/1.1
Content-Disposition: "MetalPart.meshModel"
Content-Length: 2090682
Content-MD5: "bdf6b06ab301a80ae55021085b820393"
Content-Type: "application/x-zeiss-piweb-meshmodel"
{% endhighlight %}

Response:
{% highlight http %}
HTTP/1.1 201 Created
{% endhighlight %}

####Example for webservice call via API.dll

Request:

{% highlight csharp %}
var client = new RawDataServiceRestClient( serviceUri );
//Create RawDataInformation object which contains information about the data to be uploaded
var rawDataInformation = new RawDataInformation(RawDataTargetEntity.CreateForPart(new Guid("b8f5d3fe-5bd5-406b-8053-67f647f09dc7"), -1);
rawDataInformation.MD5 = new Guid("bdf6b06ab301a80ae55021085b820393");
rawDataInformation.MimeType = "application/x-zeiss-piweb-meshmodel";
rawDataInformation.FileName = "MetalPart.meshModel";
rawDataInformation.Size = meshModelInBytes.Length;
client.CreateRawData(meshModelInBytes, rawDataInformation);
{% endhighlight %}

##Fetch raw data information

The request can be restricted by adding url parameter. For more details see the [URL-Parameter section](General-Information#raw-data).

### Fetch raw data information for a part with the uuid b8f5d3fe-5bd5-406b-8053-67f647f09dc7

#### Example for direct webservice call

Request:

{% highlight http %}
GET /rawDataServiceRest/rawData/part?uuids={b8f5d3fe-5bd5-406b-8053-67f647f09dc7} HTTP/1.1
{% endhighlight %}

Response:
{% highlight json linenos %}
{
   ...
   "data":
   [
       {
           "target":
           {
               "entity": "Part",
               "uuid": "5129295e-2605-4051-a8d5-7db57fabcab3"
           },
           "key": 0,
           "fileName": "MetalPart.meshModel",
           "description": "PiWeb Mesh Model",
           "mimeType": "application/x-zeiss-piweb-meshmodel",
           "lastModified": "2014-08-15T11:58:03.04Z",
           "created": "2014-08-15T11:58:03.04Z",
           "size": 2090682,
           "md5": "bdf6b06ab301a80ae55021085b820393"
       },
       ...
   ]
}
{% endhighlight %}

####Example for webservice call via API.dll

{% highlight csharp %}
var client = new RawDataServiceRestClient( serviceUri );
var rawDataInfo = await client.ListRawDataForParts( new Guid[] { b8f5d3fe-5bd5-406b-8053-67f647f09dc7 } );
{% endhighlight %}

##Fetch raw data 

On fetching raw data there is server side caching activated. When a raw data object is requested the first time several HTTP header values are returned beneath the raw data object. This header values include the {% highlight http %} ETag {% endhighlight %} header that consists of a distinct hash value which identifies the raw data object unambigusously. This hash value is a combination of the MD5 sum and the last modified date. If this {% highlight http %}ETag{% endhighlight %} value is sent within the {% highlight http %} If-None-Match {%endhighlight %} header on the next request the server returns a {% highlight http %} 304 - Not modified {% endhighlight %} HTTP header status code instead of the raw data object if the object has not been changed since the last request. If the API.dll is used the caching is already implemented.

### Fetch raw data with key 0 for a part with the uuid b8f5d3fe-5bd5-406b-8053-67f647f09dc7

#### Example for direct webservice call

Request:

{% highlight http %}
GET /rawDataServiceRest/rawData/part/b8f5d3fe-5bd5-406b-8053-67f647f09dc7/0 HTTP/1.1
{% endhighlight %}

Response: 

The requested raw data file

{% highlight http %}
HTTP/1.1 200 OK
Etag: "6ab0f6bd01b30aa8e55021085b820393635437006830400000"
Last-Modified: Fri, 15 Aug 2014 11:58:03 GMT
...
{% endhighlight %}

Request with If-None-Match header

{% highlight http %}
GET /rawDataServiceRest/rawData/part/b8f5d3fe-5bd5-406b-8053-67f647f09dc7/0 HTTP/1.1
If-None-Match: "6ab0f6bd01b30aa8e55021085b820393635437006830400000"
{% endhighlight %}

Response:

{% highlight http %}
HTTP/1.1 304 Not modified
{% endhighlight %}

#### Example for webservice call via API.dll

{% highlight csharp %}
var client = new RawDataServiceRestClient( serviceUri );
var rawData = await client.GetRawDataForPart( "b8f5d3fe-5bd5-406b-8053-67f647f09dc7", 0 );
{% endhighlight %}

##Update raw data

Updating a raw data object works nearly identically to adding raw data objects. The only difference is the key which the raw data object is identified by and which is mandatory.

##Delete raw data
If a key is given within the uri only the raw data object with the given key otherwise all raw data objects which belong to the entity will be deleted.

### Delete raw data object with key 0 for a part with the uuid b8f5d3fe-5bd5-406b-8053-67f647f09dc7

#### Example for direct webservice call

Request

{% highlight http %}
DELETE /rawDataServiceRest/rawData/part/b8f5d3fe-5bd5-406b-8053-67f647f09dc7/0 HTTP/1.1
{% endhighlight %}

Response

{% highlight http %}
HTTP/1.1 200 OK
{% endhighlight %}

####Example for webservice call via API.dll

Request:

{% highlight csharp %}
var client = new RawDataServiceRestClient( serviceUri );
client.DeleteRawDataForPart(new Guid("b8f5d3fe-5bd5-406b-8053-67f647f09dc7"),0);
{% endhighlight %}

### Delete all raw data objects for a part with the uuid b8f5d3fe-5bd5-406b-8053-67f647f09dc7

#### Example for direct webservice call

Request

{% highlight http %}
DELETE /rawDataServiceRest/rawData/part/b8f5d3fe-5bd5-406b-8053-67f647f09dc7  HTTP/1.1
{% endhighlight %}

Response

{% highlight http %}
HTTP/1.1 200 OK
{% endhighlight %}

####Example for webservice call via API.dll

Request:

{% highlight csharp %}
var client = new RawDataServiceRestClient( serviceUri );
client.DeleteAllRawDataForPart( new Guid("b8f5d3fe-5bd5- 406b-8053-67f647f09dc7") );
{% endhighlight %}