Eve
===

An out-of-the-box REST Web API at your finger tips. Eve will allow
you to effortlessly build and deploy a fully featured, REST-compliant,
proprietary API.

Requirements
------------
Once Eve is installed this is what you will need to bring your glorified API
online:

- A database 
- A simple configuration file
- A minimal launch script
  
Support for MongoDB comes out of the box; extensions for other SQL/NoSQL
databases can be developed with relative ease. API settings are stored in
a standard Python module that resides in your project folder (defaults to
`settings.py`), or to which you can refer by means of an environment variable.
Overall, you will find that configuring and fine-tuning your API is a very
simple process.  Most of the times the launch script will be as simple as::
    
    from eve import Eve

    app = Eve() 
    app.run()

Features
--------
- Strong emphasis on the REST 
  The Eve project aims to provide the best possibile REST-compliant API
  implementation.  Since REST is not a standard or a protocol, but rather an
  architectural style, there is really no 'golden' set of rules to follow. In
  fact, on the web you probably cannot find two APIs behaving in the exact same
  way. While designing the core API I tried to understand and follow the REST
  principles as closely as possibile.
  
- Full range of CRUD operations via HTTP verbs 
  Read-only by default, APIs support the full range of CRUD (Create, Read,
  Update, Delete) operations at both global and individual endpoint level. So
  you can have a read-only resource accessible at one endpoint along with
  a fully editable resource at another. The following table shows Eve's
  implementation of CRUD via REST

    ====== ========= ===================
    Action HTTP Verb Context 
    ====== ========= ===================
    Create POST      Collection
    Read   GET       Collection/Document
    Update PATCH     Document
    Delete DELETE    Document
    ====== ========= ===================

- Customizable resource endpoints (persistent identifiers)
  By default Eve will make known database collections available as resource
  endpoints. A `contacts` collection in the database will be made ready to be
  consumed at `api.example.com/contacts/`. You can customize the URIs of your
  resources so that, in our example, the API endpoint could become, say,
  `/customers/`. 

- Customizable, multiple resource item endpoints
  Resources can or cannot provide access to their own individual items. API
  consumers could get access to `/contacts/<ObjectId>/` and `/contacts/smith/`
  while they could only access to `/invoices/` as a whole. When you do grant
  access to resource items, you can define up to two lookup endpoints, both
  defined via regex. The first will be the primary endpoint and will match your
  database primary key structure (ie, an ObjectId in a MongoDB database). The
  second, which is optional, will better match a field with unique values,
  since Eve will retrieve only the first match anyway.

- Filtering and sorting
  Resource endpoints allow consumers to retrieve multiple documents. Query
  strings are supported, allowing for filtering and sorting when retrieving
  data. Currently two query formats are seamlessly supported: the mongo query
  syntax (`?where={"name": "john doe"}`), and the native python syntax
  (`?where=name=='john doe'`). Both query formats allow for conditional and
  logical And/Or operators, however nested and combined. Sort is currently 
  supported via standard MongoDB syntax (`?sort={"name": -1}`); support for
  a more general syntax (`?sort=name`) is planned as well.

- Paging
  Resource paging is enabled by default, in order to improve performance and
  preserve bandwith. When a consumer requests a resource, the first N items
  matching the query are serverd. Links to subsequent/previous pages are
  provided with the response. Default and maxium page size is customizable, and
  consumers can request specific pages via query string (`?page=10`).

- HATEOAS
  Hypermedia as the Engine of Application State is enabled by default. Each
  response includes a <links> section. Links provide details on their
  `relation` relative to the resource being accessed and a `title`. Titles and
  relations could be used by clients to dynamically updated their UI, or to
  navigate the API without knowing it structure beforehand. An example::

    "links": [
        <link rel='parent' title='home' href='http://api.example.com/' />,
        <link rel='collection' title='contacts' href='http://api.example.com/contacts/' />,
        <link rel='next' title='next page' href='http://api.example.com/contacts/?page=2' />,
    ]
        

- JSON and XML
  Eve responses are automatically rendered as JSON or XML depending on the
  requested `Accept` header. Inbound documents (for inserts and edits) are
  in JSON format.
  
- Last-Modified and ETag (conditional requests)
  Each resource representation provides information on the last time it was
  updated along with an hash value computed on the representation itself
  (`Last-Modified` and `ETag` response headers). These allow consumers to only
  retrieve new or modified data (`If-Modified-Since` and `If-None-Match`
  request headers).

- Data integrity and concurrency control (If-Match).
  API responses include a `ETag` header, which allows for proper concurrency
  control. An `ETag` is an hash avlue representing the current state of the
  resource on the server. Consumers are not allowed to edit or delete
  a resource unless they provide an up-to-date `ETag` for the resource they are
  attempting to edit.

- Multiple inserts
  Consumers can send a stream of multiple documents to be inserted in a given
  resource. The response will provide detailed state information about each
  item inserted (creation date, link to the item endpoint, primary key/id,
  etc.).

- Data validation
  Data validation is provided out-of-the-box. The configuration includes
  a schema definition for every resource managed by the API. Data sent to the
  API for insertion or editing will be validated against the schema and
  a resource will be updated only if validation is passed. In case of multiple
  inserts the response will provide a success/error state for each individual
  item.
  
- Extensible data validation
  Data validation is based on the Cerberus validation system and therefore it
  is extensible so you can adapt it to your specific use case. Say that your
  API can only accept odd numbers for a certain field values: you can validate
  that that. Or that you want to make sure that a VAT field actually matches
  your own country VAT algorithm: you can do that. As a matter of fact, Eve's
  MongoDB data-layer is extending Cerberus' standard validation, implementing
  the `unique` schema field constraint.

- Resource-level cache control directives 
  You can set global and individual cache-control directives for each resource.
  Directives will be included in API response headers (`Cache-Control,`
  `Expires`). This will minimize load on the server since cache-enbaled
  consumers will perform resource-intensive request only when really needed.
  to reduce load on the API.

A little context
----------------
At `Gestionale Amica<http://gestionaleamica.com>`_ we had been working hard on
a full featured, Python powered, RESTful Web API. We learned quite a few things
on REST best patterns, and we got a chance to put Python's renowned web
capabilities under review. Then at EuroPython 2012, I got a chance to share
what we learned and my talk sparked quite a bit of interest there, with several
attendees asking for more. A few months have passed since then, and still the
slides are receiving a lot of hits each day, and I keep receiving emails about
sample source code and whatnot. After all, a REST API lies in the future of
every web-oriented developer, and who isn't these days?

So I thought that perhaps I could take the proprietary, closed code (codenamed
'Adam') and refactor it "just a little bit", so that it could fit a much wider
number of use cases.  I could then release it as an open source project. Well
it turned out to be slightly more complex than that but finally here it is, and
of course it's called Eve.

It still got a long way to go before it becomes the fully featured open source,
out-of-the-box API solution I came to envision (see the Roadmap below), but
I feel that at this point the codebase is ready enough for a public preview.
This will hopefully allow for some constructive feedback and maybe, for some
contributors willing to join the ranks.

PS: the slides of my EuroPython REST API talk are `available online`_. You
might want to check them to understand why and how certain design decisions
were made, especially with regard to REST implementation.

Roadmap
-------
In no particular order, here's a partial list of the features that I plan/would
like to add to Eve, provided that there is enough interest in the project.

- Documentation (this one is coming soon!)
- Granular exception handling
- Journaling/error logging
- Server side caching
- Alternative sort syntax (`?sort=name`)
- Versioning
- Authorization (OAuth2?)
- Support for MySQL and/or other SQL/NoSQL databases

Simple live demo
----------------
For a live demo, check out the Eve-based demo API accessible at
http://eve-rest.herokuapp.com (it's on the free tier so it will probably take
a while to instantiate; successive requests will be faster). Its source code is
available at https://github.com/nicolaiarocci/eve-demo.

Installation
------------
::
    pip install eve

License
-------
Before you ask: Eve is BSD licensed! See the LICENSE for details.

Contribute
----------
Pull requests are welcome. I fully expect a number of issues to arise when
people start cannibalizing this thing. Please make sure to run the tests
(`python setup.py test`) before submitting, and to add your own tests as
needed. If you think that you can help to further develop the Eve project,
maybe by working on some of the features listed in the Roadmap, please get in
touch with me. 

.. _available online: https://speakerdeck.com/u/nicola/p/developing-restful-web-apis-with-python-flask-and-mongodb