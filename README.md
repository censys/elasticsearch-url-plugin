Elasticsearch URL Tokenizer and URL Token Filter
==============================

This plugin enables URL tokenization and token filtering by URL part.

## Compatibility

This repository has been changed to trunk-based development since we only support a single ElasticSearch version internally.
Fixes will not be backported to earlier versions, nor do we guarantee tagging each release.

### Current Version

ElasticSearch 8.5.0

## Usage
### URL Tokenizer
#### Options:
* `part`: Defaults to `null`. If left `null`, all URL parts will be tokenized, and some additional tokens (`host:port` and `protocol://host`) will be included. Can be either a string (single URL part) or an array of multiple URL parts. Options are `whole`, `protocol`, `host`, `port`, `path`, `query`, and `ref`.
* `url_decode`: Defaults to `false`. If `true`, URL tokens will be URL decoded.
* `allow_malformed`: Defaults to `false`. If `true`, malformed URLs will not be rejected, but will be passed through without being tokenized.
* `tokenize_malformed`: Defaults to `false`. Has no effect if `allow_malformed` is `false`. If both are `true`, an attempt will be made to tokenize malformed URLs using regular expressions.
* `tokenize_host`: Defaults to `true`. If `true`, the host will be further tokenized using a [reverse path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `.`.
* `tokenize_host_no_tld`: Defaults to `false`. If `true`, and used in conjunction with `tokenize_host`,  the TLD of the tokenized host will be omitted.
* `tokenize_path`: Defaults to `true`. If `true`, the path will be tokenized using a [path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `/`.
* `tokenize_query`: Defaults to `true`. If `true`, the query string will be split on `&`.

#### Example:
Index settings:
```json
{
	"settings": {
		"analysis": {
			"tokenizer": {
				"url_host": {
					"type": "url",
					"part": "host"
				}
			},
			"analyzer": {
				"url_host": {
					"tokenizer": "url_host"
				}
			}
		}
	}
}
```

Make an analysis request:
```bash
curl 'http://localhost:9200/index_name/_analyze?analyzer=url_host&pretty' -d 'https://foo.bar.com/baz.html'

{
  "tokens" : [ {
    "token" : "foo.bar.com",
    "start_offset" : 8,
    "end_offset" : 19,
    "type" : "host",
    "position" : 1
  }, {
    "token" : "bar.com",
    "start_offset" : 12,
    "end_offset" : 19,
    "type" : "host",
    "position" : 2
  }, {
    "token" : "com",
    "start_offset" : 16,
    "end_offset" : 19,
    "type" : "host",
    "position" : 3
  } ]
}
```

### URL Token Filter
#### Options:
* `part`: This option defaults to `whole`, which will cause the entire URL to be returned. In this case, the filter only serves to validate incoming URLs. Other possible values are:
`protocol`, `host`, `port`, `path`, `query`, and `ref`. Can be either a single URL part (string) or an array of URL parts.
* `url_decode`: Defaults to `false`. If `true`, the desired portion of the URL will be URL decoded.
* `allow_malformed`: Defaults to `false`. If `true`, documents containing malformed URLs will not be rejected, and an attempt will be made to parse the desired URL part from the malformed URL string.
If the desired part cannot be found, no value will be indexed for that field.
* `passthrough`: Defaults to `false`. If `true`, `allow_malformed` is implied, and any non-url tokens will be passed through the filter.  Valid URLs will be tokenized according to the filter's other settings.
* `tokenize_host`: Defaults to `true`. If `true`, the host will be further tokenized using a [reverse path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `.`.
* `tokenize_host_no_tld`: Defaults to `false`. If `true`, and used in conjunction with `tokenize_host`,  the TLD of the tokenized host will be omitted.
* `tokenize_path`: Defaults to `true`. If `true`, the path will be tokenized using a [path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `/`.
* `tokenize_query`: Defaults to `true`. If `true`, the query string will be split on `&`.

#### Example:
Set up your index like so:
```json
{
    "settings": {
        "analysis": {
            "filter": {
                "url_host": {
                    "type": "url",
                    "part": "host",
                    "url_decode": true,
                    "tokenize_host": false
                }
            },
            "analyzer": {
                "url_host": {
                    "filter": ["url_host"],
                    "tokenizer": "whitespace",
                }
            }
        }
    },
    "mappings": {
        "example_type": {
            "properties": {
                "url": {
                    "type": "multi_field",
                    "fields": {
                        "url": {"type": "string"},
                        "host": {"type": "string", "analyzer": "url_host"}
                    }
                }
            }
        }
    }
}
```

Make an analysis request:
```bash
curl 'http://localhost:9200/index_name/_analyze?analyzer=url_host&pretty' -d 'https://foo.bar.com/baz.html'

{
  "tokens" : [ {
    "token" : "foo.bar.com",
    "start_offset" : 0,
    "end_offset" : 32,
    "type" : "word",
    "position" : 1
  } ]
}
```
