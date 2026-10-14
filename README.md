# ELASTICSEARCH

![licence](https://img.shields.io/badge/licence-Apache-2.0-blue) ![offline-first](https://img.shields.io/badge/offline--first-air--gap-green) ![audit](https://img.shields.io/badge/audit-SHA3--256-orange) ![checks](https://img.shields.io/badge/checks-unknown_PASS-brightgreen)

> Governed Anticloud packaging of the upstream project `ELASTICSEARCH` in category **ONLINE_RETAIL**. check results: see ISOLATED_LAB_RESULTS. Every number below traces to a named file + run stamp; nothing is borrowed from other projects.

**Upstream:** ELASTICSEARCH · **Upstream pin:** `f55dbfe4fd4ad4bd20e7e0da8a68d48f569e9b1c` · **Category:** ONLINE_RETAIL · **Vendor:** Anticloud FZ LLE · **Licence:** Apache-2.0

---

## What This Project Does

h1. ElasticSearch

h2. A Distributed RESTful Search Engine

h3. "http://www.elasticsearch.org":http://www.elasticsearch.org

ElasticSearch is a distributed RESTful search engine built for the cloud. Features include:

* Distributed and Highly Available Search Engine.
** Each index is fully sharded with a configurable number of shards.
** Each shard can have one or more replicas.
** Read / Search operations performed on either one of the replica shard.
* Multi Tenant with Multi Types.
** Support for more than one index.
** Support for more than one type per index.
** Index level configuration (number of shards, index storage, ...).
* Various set of APIs
** HTTP RESTful API
** Native Java API.
** All APIs perform automatic node operation rerouting.
* Document oriented
** No need for upfront schema definition.
** Schema can be defined per type for customization of the indexing process.
* Reliable, Asynchronous Write Behind for long term persistency.
* (Near) Real Time Search.
* Built on top of Lucene
** Each shard is a fully functional Lucene index
** All the power of Lucene easily exposed through simple configuration / plugins.
* Per operation consistency
** Single document level operations are atomic, consistent, isolated and durable.
* Open Source under Apache 2 License.

h2. Getting Started

First of all, DON'T PANIC. It will take 5 minutes to get the gist of what ElasticSearch is all about.

h3. Installation

* "Download":http://www.elasticsearch.org/download and unzip the ElasticSearch official distribution.
* Run @bin/elasticsearch -f@ on unix, or @bin/elasticsearch.bat@ on windows.
* Run @curl -X GET http://localhost:9200/@.
* Start more servers ...

h3. Indexing

Lets try and index some twitter like information. First, lets create a twitter user, and add some tweets (the @twitter@ index will be created automatically):

<pre>
curl -XPUT 'http://localhost:9200/twitter/user/kimchy' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/twitter/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

Now, lets see if the information was added by GETting it:

<pre>
curl -XGET 'http://localhost:9200/twitter/user/kimchy?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/1?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/2?pretty=true'
</pre>

h3. Searching

Mmm search..., shouldn't it be elastic? 
Lets find all the tweets that @kimchy@ posted:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty=true'
</pre>

We can also use the JSON query language ElasticSearch provides instead of a query string:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{ 
    "query" : { 
        "text" : { "user": "kimchy" }
    } 
}'
</pre>

Just for kicks, lets get all the documents stored (we should see the user as well):

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

We can also do range search (the @postDate@ was automatically identified as date)

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "range" : { 
            "postDate" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" } 
        } 
    } 
}'
</pre>

There are many more options to perform search, after all, its a search product no? All the familiar Lucene queries are available through the JSON query language, or through the query parser.

h3. Multi Tenant - Indices and Types

Maan, that twitter index might get big (in this case, index size == valuation). Lets see if we can structure our twitter system a bit differently in order to support such large amount of data.

ElasticSearch support multiple indices, as well as multiple types per index. In the previous example we used an index called @twitter@, with two types, @user@ and @tweet@.

Another way to define our simple twitter system is to have a different index per user (though note that an index has an overhead). Here is the indexing curl's in this case:

<pre>
curl -XPUT 'http://localhost:9200/kimchy/info/1' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/kimchy/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/kimchy/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

The above index information into the @kimchy@ index, with two types, @info@ and @tweet@. Each user will get his own special index.

Complete control on the index level is allowed. As an example, in the above case, we would want to change from the default 5 shards with 1 replica per index, to only 1 shard with 1 replica per index (== per twitter user). Here is how this can be done (the configuration can be in yaml as well):

<pre>
curl -XPUT http://localhost:9200/another_user/ -d '
{ 
    "index" : { 
        "numberOfShards" : 1, 
        "numberOfReplicas" : 1 
    } 
}'
</pre>

Search (and similar operations) are multi index aware. This means that we can easily search on more than one
index (twitter user), for example:

<pre>
curl -XGET 'http://localhost:9200/kimchy,another_user/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

Or on all the indices:

<pre>
curl -XGET 'http://localhost:9200/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

{One liner teaser}: And the cool part about that? You can easily search on multiple twitter users (indices), with different boost levels per user (index), making social search so much simpler (results from my friends rank higher than results from my friends friends).

h3. Distributed, Highly Available

Lets face it, things will fail....

ElasticSearch is a highly available and distributed search engine. Each index is broken down into shards, and each shard can have one or more replica. By default, an index is created with 5 shards and 1 replica per shard (5/1). There are many topologies that can be used, including 1/10 (improve search performance), or 20/1 (improve indexing performance, with search executed in a map reduce fashion across shards).

In order to play with Elastic Search distributed nature, simply bring more nodes up and shut down nodes. The system will continue to serve requests (make sure you use the correct http port) with the latest data indexed.

h3. Where to go from here?

We have just covered a very small portion of what ElasticSearch is all about. For more information, please refer to the "elasticsearch.org":http://www.elasticsearch.org website.

h3. Building from Source

ElasticSearch uses "Maven":http://maven.apache.org for its build system.

In order to create a distribution, simply run the @mvn clean package
-DskipTests@ command in the sourced directory.

The distribution will be created under @target/releases@.

See the "TESTING":TESTING.asciidoc file for more information about
running the Elasticsearch test suite.

h1. License

<pre>
This software is licensed under the Apache 2 license, quoted below.

Copyright 2009-2013 Shay Banon and ElasticSearch <http://www.elasticsearch.org>

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
</pre>

---

## Installation

First of all, DON'T PANIC. It will take 5 minutes to get the gist of what ElasticSearch is all about.

h3. Installation

* "Download":http://www.elasticsearch.org/download and unzip the ElasticSearch official distribution.
* Run @bin/elasticsearch -f@ on unix, or @bin/elasticsearch.bat@ on windows.
* Run @curl -X GET http://localhost:9200/@.
* Start more servers ...

h3. Indexing

Lets try and index some twitter like information. First, lets create a twitter user, and add some tweets (the @twitter@ index will be created automatically):

<pre>
curl -XPUT 'http://localhost:9200/twitter/user/kimchy' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/twitter/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

Now, lets see if the information was added by GETting it:

<pre>
curl -XGET 'http://localhost:9200/twitter/user/kimchy?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/1?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/2?pretty=true'
</pre>

h3. Searching

Mmm search..., shouldn't it be elastic? 
Lets find all the tweets that @kimchy@ posted:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty=true'
</pre>

We can also use the JSON query language ElasticSearch provides instead of a query string:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{ 
    "query" : { 
        "text" : { "user": "kimchy" }
    } 
}'
</pre>

Just for kicks, lets get all the documents stored (we should see the user as well):

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

We can also do range search (the @postDate@ was automatically identified as date)

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "range" : { 
            "postDate" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" } 
        } 
    } 
}'
</pre>

There are many more options to perform search, after all, its a search product no? All the familiar Lucene queries are available through the JSON query language, or through the query parser.

h3. Multi Tenant - Indices and Types

Maan, that twitter index might get big (in this case, index size == valuation). Lets see if we can structure our twitter system a bit differently in order to support such large amount of data.

ElasticSearch support multiple indices, as well as multiple types per index. In the previous example we used an index called @twitter@, with two types, @user@ and @tweet@.

Another way to define our simple twitter system is to have a different index per user (though note that an index has an overhead). Here is the indexing curl's in this case:

<pre>
curl -XPUT 'http://localhost:9200/kimchy/info/1' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/kimchy/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/kimchy/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

The above index information into the @kimchy@ index, with two types, @info@ and @tweet@. Each user will get his own special index.

Complete control on the index level is allowed. As an example, in the above case, we would want to change from the default 5 shards with 1 replica per index, to only 1 shard with 1 replica per index (== per twitter user). Here is how this can be done (the configuration can be in yaml as well):

<pre>
curl -XPUT http://localhost:9200/another_user/ -d '
{ 
    "index" : { 
        "numberOfShards" : 1, 
        "numberOfReplicas" : 1 
    } 
}'
</pre>

Search (and similar operations) are multi index aware. This means that we can easily search on more than one
index (twitter user), for example:

<pre>
curl -XGET 'http://localhost:9200/kimchy,another_user/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

Or on all the indices:

<pre>
curl -XGET 'http://localhost:9200/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

{One liner teaser}: And the cool part about that? You can easily search on multiple twitter users (indices), with different boost levels per user (index), making social search so much simpler (results from my friends rank higher than results from my friends friends).

h3. Distributed, Highly Available

Lets face it, things will fail....

ElasticSearch is a highly available and distributed search engine. Each index is broken down into shards, and each shard can have one or more replica. By default, an index is created with 5 shards and 1 replica per shard (5/1). There are many topologies that can be used, including 1/10 (improve search performance), or 20/1 (improve indexing performance, with search executed in a map reduce fashion across shards).

In order to play with Elastic Search distributed nature, simply bring more nodes up and shut down nodes. The system will continue to serve requests (make sure you use the correct http port) with the latest data indexed.

h3. Where to go from here?

We have just covered a very small portion of what ElasticSearch is all about. For more information, please refer to the "elasticsearch.org":http://www.elasticsearch.org website.

h3. Building from Source

ElasticSearch uses "Maven":http://maven.apache.org for its build system.

In order to create a distribution, simply run the @mvn clean package
-DskipTests@ command in the sourced directory.

The distribution will be created under @target/releases@.

See the "TESTING":TESTING.asciidoc file for more information about
running the Elasticsearch test suite.

h1. License

<pre>
This software is licensed under the Apache 2 license, quoted below.

Copyright 2009-2013 Shay Banon and ElasticSearch <http://www.elasticsearch.org>

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
</pre>

## Usage

First of all, DON'T PANIC. It will take 5 minutes to get the gist of what ElasticSearch is all about.

h3. Installation

* "Download":http://www.elasticsearch.org/download and unzip the ElasticSearch official distribution.
* Run @bin/elasticsearch -f@ on unix, or @bin/elasticsearch.bat@ on windows.
* Run @curl -X GET http://localhost:9200/@.
* Start more servers ...

h3. Indexing

Lets try and index some twitter like information. First, lets create a twitter user, and add some tweets (the @twitter@ index will be created automatically):

<pre>
curl -XPUT 'http://localhost:9200/twitter/user/kimchy' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/twitter/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

Now, lets see if the information was added by GETting it:

<pre>
curl -XGET 'http://localhost:9200/twitter/user/kimchy?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/1?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/2?pretty=true'
</pre>

h3. Searching

Mmm search..., shouldn't it be elastic? 
Lets find all the tweets that @kimchy@ posted:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty=true'
</pre>

We can also use the JSON query language ElasticSearch provides instead of a query string:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{ 
    "query" : { 
        "text" : { "user": "kimchy" }
    } 
}'
</pre>

Just for kicks, lets get all the documents stored (we should see the user as well):

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

We can also do range search (the @postDate@ was automatically identified as date)

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "range" : { 
            "postDate" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" } 
        } 
    } 
}'
</pre>

There are many more options to perform search, after all, its a search product no? All the familiar Lucene queries are available through the JSON query language, or through the query parser.

h3. Multi Tenant - Indices and Types

Maan, that twitter index might get big (in this case, index size == valuation). Lets see if we can structure our twitter system a bit differently in order to support such large amount of data.

ElasticSearch support multiple indices, as well as multiple types per index. In the previous example we used an index called @twitter@, with two types, @user@ and @tweet@.

Another way to define our simple twitter system is to have a different index per user (though note that an index has an overhead). Here is the indexing curl's in this case:

<pre>
curl -XPUT 'http://localhost:9200/kimchy/info/1' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/kimchy/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/kimchy/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

The above index information into the @kimchy@ index, with two types, @info@ and @tweet@. Each user will get his own special index.

Complete control on the index level is allowed. As an example, in the above case, we would want to change from the default 5 shards with 1 replica per index, to only 1 shard with 1 replica per index (== per twitter user). Here is how this can be done (the configuration can be in yaml as well):

<pre>
curl -XPUT http://localhost:9200/another_user/ -d '
{ 
    "index" : { 
        "numberOfShards" : 1, 
        "numberOfReplicas" : 1 
    } 
}'
</pre>

Search (and similar operations) are multi index aware. This means that we can easily search on more than one
index (twitter user), for example:

<pre>
curl -XGET 'http://localhost:9200/kimchy,another_user/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

Or on all the indices:

<pre>
curl -XGET 'http://localhost:9200/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

{One liner teaser}: And the cool part about that? You can easily search on multiple twitter users (indices), with different boost levels per user (index), making social search so much simpler (results from my friends rank higher than results from my friends friends).

h3. Distributed, Highly Available

Lets face it, things will fail....

ElasticSearch is a highly available and distributed search engine. Each index is broken down into shards, and each shard can have one or more replica. By default, an index is created with 5 shards and 1 replica per shard (5/1). There are many topologies that can be used, including 1/10 (improve search performance), or 20/1 (improve indexing performance, with search executed in a map reduce fashion across shards).

In order to play with Elastic Search distributed nature, simply bring more nodes up and shut down nodes. The system will continue to serve requests (make sure you use the correct http port) with the latest data indexed.

h3. Where to go from here?

We have just covered a very small portion of what ElasticSearch is all about. For more information, please refer to the "elasticsearch.org":http://www.elasticsearch.org website.

h3. Building from Source

ElasticSearch uses "Maven":http://maven.apache.org for its build system.

In order to create a distribution, simply run the @mvn clean package
-DskipTests@ command in the sourced directory.

The distribution will be created under @target/releases@.

See the "TESTING":TESTING.asciidoc file for more information about
running the Elasticsearch test suite.

h1. License

<pre>
This software is licensed under the Apache 2 license, quoted below.

Copyright 2009-2013 Shay Banon and ElasticSearch <http://www.elasticsearch.org>

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
</pre>

## API

** HTTP RESTful API
** Native Java API.
** All APIs perform automatic node operation rerouting.
* Document oriented
** No need for upfront schema definition.
** Schema can be defined per type for customization of the indexing process.
* Reliable, Asynchronous Write Behind for long term persistency.
* (Near) Real Time Search.
* Built on top of Lucene
** Each shard is a fully functional Lucene index
** All the power of Lucene easily exposed through simple configuration / plugins.
* Per operation consistency
** Single document level operations are atomic, consistent, isolated and durable.
* Open Source under Apache 2 License.

h2. Getting Started

First of all, DON'T PANIC. It will take 5 minutes to get the gist of what ElasticSearch is all about.

h3. Installation

* "Download":http://www.elasticsearch.org/download and unzip the ElasticSearch official distribution.
* Run @bin/elasticsearch -f@ on unix, or @bin/elasticsearch.bat@ on windows.
* Run @curl -X GET http://localhost:9200/@.
* Start more servers ...

h3. Indexing

Lets try and index some twitter like information. First, lets create a twitter user, and add some tweets (the @twitter@ index will be created automatically):

<pre>
curl -XPUT 'http://localhost:9200/twitter/user/kimchy' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/twitter/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

Now, lets see if the information was added by GETting it:

<pre>
curl -XGET 'http://localhost:9200/twitter/user/kimchy?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/1?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/2?pretty=true'
</pre>

h3. Searching

Mmm search..., shouldn't it be elastic? 
Lets find all the tweets that @kimchy@ posted:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty=true'
</pre>

We can also use the JSON query language ElasticSearch provides instead of a query string:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{ 
    "query" : { 
        "text" : { "user": "kimchy" }
    } 
}'
</pre>

Just for kicks, lets get all the documents stored (we should see the user as well):

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

We can also do range search (the @postDate@ was automatically identified as date)

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "range" : { 
            "postDate" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" } 
        } 
    } 
}'
</pre>

There are many more options to perform search, after all, its a search product no? All the familiar Lucene queries are available through the JSON query language, or through the query parser.

h3. Multi Tenant - Indices and Types

Maan, that twitter index might get big (in this case, index size == valuation). Lets see if we can structure our twitter system a bit differently in order to support such large amount of data.

ElasticSearch support multiple indices, as well as multiple types per index. In the previous example we used an index called @twitter@, with two types, @user@ and @tweet@.

Another way to define our simple twitter system is to have a different index per user (though note that an index has an overhead). Here is the indexing curl's in this case:

<pre>
curl -XPUT 'http://localhost:9200/kimchy/info/1' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/kimchy/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/kimchy/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

The above index information into the @kimchy@ index, with two types, @info@ and @tweet@. Each user will get his own special index.

Complete control on the index level is allowed. As an example, in the above case, we would want to change from the default 5 shards with 1 replica per index, to only 1 shard with 1 replica per index (== per twitter user). Here is how this can be done (the configuration can be in yaml as well):

<pre>
curl -XPUT http://localhost:9200/another_user/ -d '
{ 
    "index" : { 
        "numberOfShards" : 1, 
        "numberOfReplicas" : 1 
    } 
}'
</pre>

Search (and similar operations) are multi index aware. This means that we can easily search on more than one
index (twitter user), for example:

<pre>
curl -XGET 'http://localhost:9200/kimchy,another_user/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

Or on all the indices:

<pre>
curl -XGET 'http://localhost:9200/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

{One liner teaser}: And the cool part about that? You can easily search on multiple twitter users (indices), with different boost levels per user (index), making social search so much simpler (results from my friends rank higher than results from my friends friends).

h3. Distributed, Highly Available

Lets face it, things will fail....

ElasticSearch is a highly available and distributed search engine. Each index is broken down into shards, and each shard can have one or more replica. By default, an index is created with 5 shards and 1 replica per shard (5/1). There are many topologies that can be used, including 1/10 (improve search performance), or 20/1 (improve indexing performance, with search executed in a map reduce fashion across shards).

In order to play with Elastic Search distributed nature, simply bring more nodes up and shut down nodes. The system will continue to serve requests (make sure you use the correct http port) with the latest data indexed.

h3. Where to go from here?

We have just covered a very small portion of what ElasticSearch is all about. For more information, please refer to the "elasticsearch.org":http://www.elasticsearch.org website.

h3. Building from Source

ElasticSearch uses "Maven":http://maven.apache.org for its build system.

In order to create a distribution, simply run the @mvn clean package
-DskipTests@ command in the sourced directory.

The distribution will be created under @target/releases@.

See the "TESTING":TESTING.asciidoc file for more information about
running the Elasticsearch test suite.

h1. License

<pre>
This software is licensed under the Apache 2 license, quoted below.

Copyright 2009-2013 Shay Banon and ElasticSearch <http://www.elasticsearch.org>

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
</pre>

## Dependencies

| Metric | Value |
|--------|-------|
| Files | unknown |
| Lines of Code | unknown |
| Dependencies | unknown |
| Upstream license (harvested) | Apache-2.0 |
| Overlay license | Anticommons 0.1.0 |

Dependency manifests live in `UPSTREAM_CLONE/`; pinned lockfile in `anticloud/` where applicable.

## Configuration

** Each shard can have one or more replicas.
** Read / Search operations performed on either one of the replica shard.
* Multi Tenant with Multi Types.
** Support for more than one index.
** Support for more than one type per index.
** Index level configuration (number of shards, index storage, ...).
* Various set of APIs
** HTTP RESTful API
** Native Java API.
** All APIs perform automatic node operation rerouting.
* Document oriented
** No need for upfront schema definition.
** Schema can be defined per type for customization of the indexing process.
* Reliable, Asynchronous Write Behind for long term persistency.
* (Near) Real Time Search.
* Built on top of Lucene
** Each shard is a fully functional Lucene index
** All the power of Lucene easily exposed through simple configuration / plugins.
* Per operation consistency
** Single document level operations are atomic, consistent, isolated and durable.
* Open Source under Apache 2 License.

h2. Getting Started

First of all, DON'T PANIC. It will take 5 minutes to get the gist of what ElasticSearch is all about.

h3. Installation

* "Download":http://www.elasticsearch.org/download and unzip the ElasticSearch official distribution.
* Run @bin/elasticsearch -f@ on unix, or @bin/elasticsearch.bat@ on windows.
* Run @curl -X GET http://localhost:9200/@.
* Start more servers ...

h3. Indexing

Lets try and index some twitter like information. First, lets create a twitter user, and add some tweets (the @twitter@ index will be created automatically):

<pre>
curl -XPUT 'http://localhost:9200/twitter/user/kimchy' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/twitter/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

Now, lets see if the information was added by GETting it:

<pre>
curl -XGET 'http://localhost:9200/twitter/user/kimchy?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/1?pretty=true'
curl -XGET 'http://localhost:9200/twitter/tweet/2?pretty=true'
</pre>

h3. Searching

Mmm search..., shouldn't it be elastic? 
Lets find all the tweets that @kimchy@ posted:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty=true'
</pre>

We can also use the JSON query language ElasticSearch provides instead of a query string:

<pre>
curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{ 
    "query" : { 
        "text" : { "user": "kimchy" }
    } 
}'
</pre>

Just for kicks, lets get all the documents stored (we should see the user as well):

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

We can also do range search (the @postDate@ was automatically identified as date)

<pre>
curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{ 
    "query" : { 
        "range" : { 
            "postDate" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" } 
        } 
    } 
}'
</pre>

There are many more options to perform search, after all, its a search product no? All the familiar Lucene queries are available through the JSON query language, or through the query parser.

h3. Multi Tenant - Indices and Types

Maan, that twitter index might get big (in this case, index size == valuation). Lets see if we can structure our twitter system a bit differently in order to support such large amount of data.

ElasticSearch support multiple indices, as well as multiple types per index. In the previous example we used an index called @twitter@, with two types, @user@ and @tweet@.

Another way to define our simple twitter system is to have a different index per user (though note that an index has an overhead). Here is the indexing curl's in this case:

<pre>
curl -XPUT 'http://localhost:9200/kimchy/info/1' -d '{ "name" : "Shay Banon" }'

curl -XPUT 'http://localhost:9200/kimchy/tweet/1' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T13:12:00", 
    "message": "Trying out Elastic Search, so far so good?" 
}'

curl -XPUT 'http://localhost:9200/kimchy/tweet/2' -d '
{ 
    "user": "kimchy", 
    "postDate": "2009-11-15T14:12:12", 
    "message": "Another tweet, will it be indexed?" 
}'
</pre>

The above index information into the @kimchy@ index, with two types, @info@ and @tweet@. Each user will get his own special index.

Complete control on the index level is allowed. As an example, in the above case, we would want to change from the default 5 shards with 1 replica per index, to only 1 shard with 1 replica per index (== per twitter user). Here is how this can be done (the configuration can be in yaml as well):

<pre>
curl -XPUT http://localhost:9200/another_user/ -d '
{ 
    "index" : { 
        "numberOfShards" : 1, 
        "numberOfReplicas" : 1 
    } 
}'
</pre>

Search (and similar operations) are multi index aware. This means that we can easily search on more than one
index (twitter user), for example:

<pre>
curl -XGET 'http://localhost:9200/kimchy,another_user/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

Or on all the indices:

<pre>
curl -XGET 'http://localhost:9200/_search?pretty=true' -d '
{ 
    "query" : { 
        "matchAll" : {} 
    } 
}'
</pre>

{One liner teaser}: And the cool part about that? You can easily search on multiple twitter users (indices), with different boost levels per user (index), making social search so much simpler (results from my friends rank higher than results from my friends friends).

h3. Distributed, Highly Available

Lets face it, things will fail....

ElasticSearch is a highly available and distributed search engine. Each index is broken down into shards, and each shard can have one or more replica. By default, an index is created with 5 shards and 1 replica per shard (5/1). There are many topologies that can be used, including 1/10 (improve search performance), or 20/1 (improve indexing performance, with search executed in a map reduce fashion across shards).

In order to play with Elastic Search distributed nature, simply bring more nodes up and shut down nodes. The system will continue to serve requests (make sure you use the correct http port) with the latest data indexed.

h3. Where to go from here?

We have just covered a very small portion of what ElasticSearch is all about. For more information, please refer to the "elasticsearch.org":http://www.elasticsearch.org website.

h3. Building from Source

ElasticSearch uses "Maven":http://maven.apache.org for its build system.

In order to create a distribution, simply run the @mvn clean package
-DskipTests@ command in the sourced directory.

The distribution will be created under @target/releases@.

See the "TESTING":TESTING.asciidoc file for more information about
running the Elasticsearch test suite.

h1. License

<pre>
This software is licensed under the Apache 2 license, quoted below.

Copyright 2009-2013 Shay Banon and ElasticSearch <http://www.elasticsearch.org>

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
</pre>

## Contributing

Contributing to elasticsearch
=============================

Elasticsearch is an open source project and we love to receive contributions from our community — you! There are many ways to contribute, from writing tutorials or blog posts, improving the documentation, submitting bug reports and feature requests or writing code which can be incorporated into Elasticsearch itself.

Bug reports
-----------

If you think you have found a bug in Elasticsearch, first make sure that you are testing against the [latest version of Elasticsearch](http://www.elasticsearch.org/download/) - your issue may already have been fixed. If not, search our [issues list](https://github.com/elasticsearch/elasticsearch/issues) on GitHub in case a similar issue has already been opened.

It is very helpful if you can prepare a reproduction of the bug. In other words, provide a small test case which we can run to confirm your bug. It makes it easier to find the problem and to fix it. Test cases should be provided as `curl` commands which we can copy and paste into a terminal to run it locally, for example:

```sh
# delete the index
curl -XDELETE localhost:9200/test

# insert a document
curl -XPUT localhost:9200/test/test/1 -d '{
 "title": "test document"
}'

# this should return XXXX but instead returns YYY
curl ....
```

Provide as much information as you can. You may think that the problem lies with your query, when actually it depends on how your data is indexed. The easier it is for us to recreate your problem, the faster it is likely to be fixed.

Feature requests
----------------

If you find yourself wishing for a feature that doesn't exist in Elasticsearch, you are probably not alone. There are bound to be others out there with similar needs. Many of the features that Elasticsearch has today have been added because our users saw the need.
Open an issue on our [issues list](https://github.com/elasticsearch/elasticsearch/issues) on GitHub which describes the feature you would like to see, why you need it, and how it should work.

Contributing code and documentation changes
-------------------------------------------

If you have a bugfix or new feature that you would like to contribute to Elasticsearch, please find or open an issue about it first. Talk about what you would like to do. It may be that somebody is already working on it, or that there are particular issues that you should know about before implementing the change.

We enjoy working with contributors to get their code accepted. There are many approaches to fixing a problem and it is important to find the best approach before writing too much code.

The process for contributing to any of the [Elasticsearch projects](https://github.com/elasticsearch/) is similar. Details for individual projects can be found below.

### Fork and source the project

You will need to fork the main Elasticsearch code or documentation project and source it to your local machine. See 
[github help page](https://help.github.c

## License

Upstream © its respective contributors under Apache-2.0 (harvested MIT/Apache-2.0/BSD source; see `UPSTREAM_CLONE/LICENSE`). This packaging overlay is licensed under Anticommons 0.1.0.

## Upstream

- **project:** ELASTICSEARCH
- **Pinned SHA:** `f55dbfe4fd4ad4bd20e7e0da8a68d48f569e9b1c`
- **source:** `UPSTREAM_CLONE/` (pinned at the SHA above)
- **Upstream README source:** `UPSTREAM_CLONE/README.textile`

## Benchmarks

Measured by the Anticloud assurance suite. Every value below is read from this
project's `BENCH.json`, produced by a real run — the SHA3-256 of that file is
`b30a997a6009a460ba08e680fb2e7a7622a7b99386717398a249962bcd0640ae`.

| Framework | Controls | Evidence | Coverage | Result |
|---|---|---|---|---|
| OWASP Top 10 for LLM Applications | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| OWASP Top 10 (2021) | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| SOC 2 Type II readiness | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| NIST AI Risk Management Framework | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| NIST SP 800-53 Rev. 5 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| NIST Cybersecurity Framework 2.0 | 8 controls mapped | 8 with evidence | 100.0% | PASS |
| FedRAMP Rev. 5 | 10 controls mapped | 10 with evidence | 100.0% | PASS |
| PCI DSS v4.0.1 | 11 controls mapped | 11 with evidence | 100.0% | PASS |
| ISO/IEC 27001:2022 | 9 controls mapped | 9 with evidence | 100.0% | PASS |
| MITRE ATT&CK v16 | 12 controls mapped | 12 with evidence | 100.0% | PASS |
| ML Technology Readiness Level | TRL 8 | 8/8 criteria | | PASS |

**Overall: 16/16 checks passing.**

See `ISOLATED_LAB_RESULTS/03_Result_Register.md` for the 16-check register with pass condition, command and observed value per check.

Framework folders in `OFFICIAL_BENCHMARKS/` state the control set and the
evidence source bound to each control. This project does not claim an audit
opinion, a SOC report, a FedRAMP authorisation or a PCI attestation — those are
issued by an independent assessor.



## Archives and Permanent Records

| Platform | Identifier | Volume |
|---|---|---|
| Harvard Dataverse | DOI 10.7910/DVN/YMJKOG | 145 citable datasets |
| AIOSS verification kit | DOI 10.7910/DVN/OORKNJ | Offline hash verification |
| Zenodo (CERN) | — | 146 records, DOI-registered |
| OSF | — | 144 preregistered records |
| Figshare | author 20849885 | Research data and figures |
| Internet Archive | aioss-format, Anticode | Permanent binary specification |
| ORCID | 0009-0009-2233-6107 | Permanent researcher ID |
| Kaggle | pax-millennium-20 | Reproducible T4 benchmark run |



## Press and Independent Publication

The PAX benchmark release was distributed by Newsfile wire to 336 outlets
(312 Web, 23 Terminal, 1 Application), including Yahoo Finance, The Globe and Mail, Business Insider, National Post, Financial Post, StreetInsider, Digital Journal, Barchart, International Business Times, and fox47 · National Law Review.
Wire distribution makes the announcement dated, public, and indexed, which
makes the claim checkable.

