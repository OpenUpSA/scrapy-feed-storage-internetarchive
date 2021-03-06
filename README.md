# scrapy-feed-storage-internetarchive

This is a [Storage Backend](https://docs.scrapy.org/en/latest/topics/feed-exports.html#storages) for [Scrapy](https://scrapy.org/) [Item Feeds](https://docs.scrapy.org/en/latest/topics/feed-exports.html) that uploads feed files to [archive.org](https://archive.org) when a scrape job ends.

This was created to make it easy to archive data at the Internet Archive which you are authorised to distribute. e.g. to archive public data.


## Usage

### Install the custom storage backend

We recommend the scheme `internetarchive`

```
FEED_STORAGES = {
    "internetarchive": "feedstorage_internetarchive.storages.InternetArchiveStorage",
}
```


### Configure the Internet Archive metadata template

Metadata values can be specified using the settings key `FEED_STORAGE_INTERNETARCHIVE`, e.g.

```
FEED_STORAGE_INTERNETARCHIVE = {
    "metadata": {
        "mediatype": "data",
        "coverage": "South Africa",
        "title": "eTender Portal %(name)s %(time)s %(filetype)s",
    }
}
```


### Configure the storage for your feeds

Use the Feed Exporter configuration with the URI scheme you used for installing the backend.

The Internet Archive feed exporter should have the hostname `archive.org`, Internet Archive S3 API access key and secret in the username and password positions.

Only one level of path is allowed. This will be used as the filename, and will be transformed into a unique identifier, meaning it should be unique on all of the Internet Archive. Including the scrape job timestamp in this path is useful for ensuring uniqueness.

Extra parameters can be provided as query string parameters, which will then be templated into the metadata values.

e.g.

```
FEEDS = {
    "internetarchive://YourIAS3AccessKey:YourIAS3APISecretKey@archive.org/south-africa-%(name)s-%(time)s.csv?time=%(time)s&name=%(name)s&filetype=csv": {
        "format": "csv",
    },
    "internetarchive://YourIAS3AccessKey:YourIAS3APISecretKey@archive.org/south-africa-%(name)s-%(time)s.jsonlines?time=%(time)s&name=%(name)s&filetype=jsonlines": {
        "format": "jsonlines",
    },
}
```

You probably don't want to put credentials into your project settings module, since it can then easily be discovered if added to source control. So try to set it in the environment where you will run your spider.


### Scrapinghub

You can set the `FEEDS` key in scrapinghub by providing the value dictionary as JSON on a single line in your spider's Raw Settings. For the above example, you would add the following line in the scrapinghub Raw settings:

```
FEEDS = {"internetarchive://YourIAS3AccessKey:YourIAS3APISecretKey@archive.org/south-africa-%(name)s-%(time)s.csv?time=%(time)s&name=%(name)s": {"format": "csv"}, "internetarchive://YourIAS3AccessKey:YourIAS3APISecretKey@archive.org/south-africa-%(name)s-%(time)s.jsonlines?time=%(time)s&name=%(name)s": { "format": "jsonlines" }}
```

After saving, you should see it parsed into a key and value on the standard settings pane.
