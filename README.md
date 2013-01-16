KPIggyBank
==========

A backend store for Key Performance Indicator (KPI) data.

Node.JS + CouchDB.

Piggybanks have a somewhat peculiar protocol in that they are write-many read-once.  You can put as many blobs (coins, bills, gift cards, it doesn't care because it's document-centric) as you want into piggybank (within allocated storage of course), but the extraction of resources from piggybank (coins, etc) is a one-time destructive operation (what old-timers referred to as "smashing it on the floor").

Requirements
------------

- An accessible CouchDB server for persistence.
- Node.JS (0.6.17 or greater)

Installation
------------

- git clone: `https://github.com/mozilla/kpiggybank`
- `npm install`

Testing
-------

- `npm test`

The test suite simulates throwing a thousand login sequences at the KPI store.

It is anticipated that, with 1 million users, BrowserID will generate some 
100 sign-in activities per second.  The test suite requires that kpiggy bank
can completely store and retrieve records at a rate at least twice as fast
as this.

If you want to experiment with the server without having couch installed, 
use the in-memory data store:

    DB_BACKEND=memory node lib/server.js

Note that the in-memory data is not saved anywhere.  It's just for testing.

Running
-------

For configuration, the file `env.sh.dist` can be copied to `env.sh` and edited.
kpiggybank will look for the following environment variables:

- `DB_BACKEND`: One of "couchdb", "memory", "dummy". Default "couchdb".
- `DB_HOST`: IP addr of couch server.  Default "127.0.0.1".
- `DB_PORT`: Port number of couch server.  Default "5984".
- `DB_NAME`: Name of the database.  Default "bid_kpi".
- `DB_USER`: Username for database if required.  Default "kpiggybank".
- `DB_PASS`: Password for database if required.  Default "kpiggybank".
- `HOST`: "127.0.0.1"
- `PORT`: Port for the kpiggybank server.  Default "3000".
- `MODE`: Governs how verbose logging should be.  Set to "prod" for quieter logging.  Default "dev".

Start the server like so:

- `npm start`


Or like so:

- `node lib/server.js`

Or change your env configuration with something like:

- `DB_NAME=bid_kpi_test npm start`

When running kpiggybank for the first time on a given database, it will 
ensure that the db exists, creating it if it doesn't.

Please note that the database named `bid_kpi_test` is **deleted** as part of the 
test suite.

Running on AWS
--------------

You can use in-tree awsbox scripts to deploy kpiggybank on Amazon's cloud infrastructure.

Clone kpiggybank:

    $ git clone https://github.com/mozilla/kpiggybank
    
Install node modules:
    
    $ npm install

You'll need an AWS account "signed up" for EC2. To set up AWS credentials for the scripts to access, you might put something like this
in your `.bashrc`:

    # This is your Access Key ID from your AWS Security Credentials
    export AWS_ID=<your id>
    # This is your Secret Access Key from your AWS Security Credentials
    export AWS_SECRET=<your secret>

Make sure you have an ssh key in ~/.ssh/id_rsa.pub

Now you can run the awsbox script, which will fire up an AWS instance and install couchdb. Choose a name for your VM:

    $ node_modules/.bin/awsbox create -n myvm

Once it's ready, push the kpiggybank code to the server (this will trigger awsbox to startup kpiggybank):

    $ git push myvm HEAD:master

Now `http://<aws-ip>` should point to your new kpiggybank.

For more information about awsbox: https://github.com/mozilla/awsbox

JS API
------

### Methods

- `api.saveData(blob [, callback])` - save a hopefully valid event blob
- `api.fetchRange([ options, ] callback)` - fetch some or all events
- `api.count(callback)` - get number of records in DB
- `api.followChanges()` - connect to event stream

### Events

- `change` - a newly-arrived json blob of delicious KPI data
- `error` - oh noes

### Examples

The HTTP API calls are wrapped for convenience in a JS module.  You can of 
course call the HTTP methods directly if you want.  Example of using the JS
API:

``` js
    var API = require("lib/api");
    var api = new API(server_host, server_port);
    api.saveData(yourblob, yourcallback);
```

The callback is optional.

To query a range:

``` js
    var options = {start: 1, end: 42}; // optional 
    api.fetchRange(options, callback);
```

options are ... optional, so you can get all records like so:

``` js
    api.fetchRange(callback);
```

Subscribe to changes stream.  The changes stream is an event emitter.  Use like
so:

``` js
    api.followChanges()  // now subscribed

    api.on('change', function(change) {
        // do something visually stunning
    });
```


HTTP API
--------

*Post Data*

Post a blob of data to `/wsapi/interaction_data`.  
The post `data` should contain a JSON object following
the example here: 
https://wiki.mozilla.org/Privacy/Reviews/KPI_Backend#Example_data

In particular, the `timestamp` field is required, and should be a unix 
timestamp (seconds since the epoch); not an ISO date string.

- url: `/wsapi/interaction_data`
- method: POST
- required param: `{data: <your data blob>}`

*Get Data*

Retrieve a range of records; returns a JSON string.

- url: `/wsapi/interaction_data?start=<date-start>&end=<date-end>`
- method: GET

*Count Records*

Retrieve a count of the number of records; returns a JSON encoded number.

- url: `/wsapi/interaction_data/count`
- method: GET

License
-------

All source code here is available under the [MPL 2.0][] license, unless
otherwise indicated.

[MPL 2.0]: https://mozilla.org/MPL/2.0/


