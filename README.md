# TwtrTrkr

[TwtrTrkr.xyz](twtrtrkr.xyz) was designed to allow real-time tracking of Twitter users' sentiment. The application implements a distributed real-time streaming querying system that is engineered to scale to large input streams and large volumes of queries.

While in this case the architecture was employed to track aggregate Twitter users' sentiment (which could have many potential business & market research applications), this design could easily be re-purposed to perform large scale parsing or filtering of almost any real-time source against voluminous queries.

## Pipeline

### Technologies

* [Apache Kafka](http://kafka.apache.org)
* [Apache Storm](http://storm.apache.org)
* [Lucene Luwak](https://github.com/flaxsearch/luwak)
* [NLTK (VADER)](https://github.com/cjhutto/vaderSentiment)
* [Redis](http://redis.io)

For the purposes of my application I decided to use Apache Kafka to manage ingestion of the Twitter Streaming API as well as the user generated queries. Storm then consumes messages from Kafka shuffling new queries across the distributed query indicies (implemented by Luwak) or broadcasted tweets through the virtual query index. Matching tweets are then processed by the NLTK tool, VADER, to generate a sentiment rating before being aggregated by query in a final layer of Storm bolts. Ultimately the aggregated user sentiments are passed from Storm through a Redis PubSub framework before being displayed to each user.

![Pipeline](https://github.com/zubairsaiyed/TwtrTrkr/blob/master/images/pipeline.png)

## Implementation

This source code for this project is contained across three modules:

* [tweet-stream](https://github.com/zubairsaiyed/tweet-stream/)
 * Kafka producers connecting to the Twitter API Stream
* [tweet-storm](https://github.com/zubairsaiyed/tweet-storm/)
 * The storm topology integrating with Lucene Luwak and NLTK VADER
* [tweet-ui](https://github.com/zubairsaiyed/tweet-ui/)
 * Web framework which submits user queries to Kafka, connects to the Redis PubSub framework, and displays results to the user

Details regarding each submodule's implementation can be found on their respective github pages.

## Design Considerations

In order to implement stream searching, I opted to use Lucene Luwak as opposed to the more popular [Elasticsearch Percolator](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-percolate.html), to leverage it's presearcher functionality leading to significant performance advantages (between 6-40x improvements according to some [studies](http://www.flax.co.uk/blog/2015/07/27/a-performance-comparison-of-streamed-search-implementations/)).

However, Luwak as-is is not capable of handling arbitrarily large numbers of queries - it is constrained by the available memory of the system on which it is installed. To overcome this limitation I leveraged Apache Storm to design a topology that distributes the tracked queries across multiple Luwak bolts and intelligently routes incoming tweets through an effective 'virtual inverted query index'.

## Technical Presentation

A brief technical presentation on the design of this system is accessible [here](http://bit.do/twtrtrkr).
