#A whole Wikipedia, right in mongodb
put a epic **zillion-Gb** [wikipedia dump](https://dumps.wikimedia.org) quickly into mongo, with fully-parsed wikiscript, without thinking, without loading it into memory, grepping, unzipping, or other cli nonsense.

It's a javascript-only one-liner that can get a highly-queryable wikipedia on your laptop in a nice afternoon.

It also uses [wtf_wikipedia](https://github.com/spencermountain/wtf_wikipedia) to parse wikiscript into *nearly-nice* json.

```bash
npm install -g wikipedia-to-mongodb
wikipedia-to-mongodb /path/to/my-wikipedia-article-dump.xml.bz2
```
then,
````javascript
db.wikipedia.find({title:"Toronto"})[0].categories
//[ "Former colonial capitals in Canada",
//  "Populated places established in 1793",
//  ...]
db.wikipedia.count({type:"redirect"})
// 124,999...
````

this library uses:
* [unbzip2-stream](https://github.com/regular/unbzip2-stream) to stream-uncompress the gnarly bz2 file

* [xml-stream](https://github.com/assistunion/xml-stream) to stream-parse its xml format

* [wtf_wikipedia](https://github.com/spencermountain/wtf_wikipedia) to brute-parse the article wikiscript contents into **almost-pretty** JSON.

* [redis](http://redis.io/) to (optionally) put wikiscript parsing on separate threads :metal:

# #1
you can do this.
a few Gb. you can do this.
you can do this.
# #2 get ready
Install [nodejs](https://nodejs.org/en/), [mongodb](https://docs.mongodb.com/manual/installation/), and optionally [redis](http://redis.io/)

`git clone git@github.com:spencermountain/wikipedia-to-mongodb.git`
`cd wikipedia-to-mongodb`
`npm install`

# #3 download wikipedia
The Afrikaans wikipedia (only 33 556 artikels) only takes a few minutes to download, and 10 mins to load into mongo on a macbook.
```bash
# dowload an xml dump (38mb, couple minutes)
wget https://dumps.wikimedia.org/afwiki/latest/afwiki-latest-pages-articles.xml.bz2
```
use whichever xml dump you'd like.

# #4 get going
```bash
#load it into mongo (10-15 minutes)
wikipedia-to-mongodb ./afwiki-latest-pages-articles.xml.bz2
```

# #5 check out your data
to view your data in the mongo console,
````javascript
$ mongo
use af_wikipedia

//shows a random page
db.wikipedia.find().skip(200).limit(2)

//count the redirects (~5,000 in afrikaans)
db.wikipedia.count({type:"redirect"})

//find a specific page
db.wikipedia.findOne({title:"Toronto"}).categories
````


##Same for the English wikipedia:
the english wikipedia will work under the same process, but
the download will take an afternoon, and the loading/parsing a couple hours. The en wikipedia dump is a 4gb download and becomes a pretty legit mongo collection uncompressed. It's something like 40gb, but mongo can do it... You can do it!

## --worker flag
there is yet much faster way (even x10) to import all pages into mongodb but a little more complex. it requires redis installed on your computer and running worker in separate process
````bash
# install redis on ubuntu
sudo apt-get install redis-server

# clone wikipedia-to-mongodb
git clone git@github.com:spencermountain/wikipedia-to-mongodb.git

#load pages into job queue (there as additional param -w)
node index.js ./afwiki-latest-pages-articles.xml.bz2 -w

# start processing jobs (parsing articles and saving to mongodb) on all CPU's
node worker.js

# you can preview processing jobs in kue dashboard (localhost:3000)
node node_modules/kue/bin/kue-dashboard -p 3000
````
