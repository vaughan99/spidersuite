# Spidersuite

This project uses Node.js to implement a spider, and outputs a list of warnings, errors, and 404s.

* [Install spidersuite](#install-spidersuite)
* [Usage](#usage)
* [Run spidersuite](#run-spidersuite)
* [Broken links](#broken-links)
* [Broken redirects](#broken-redirects)
* [Config](#config)

## Install spidersuite

```
$ npm install spidersuite --save-dev
```

## Usage

1. Start the app that you want to crawl.

1. Open another Terminal window in the same directory as your project.
    Run node for your localhost:
    ```
    ./node_modules/.bin/spidersuite [--config <path to config>] https://localhost:8443/
    ```

    The spidersuite [results](#broken-links) appear in this Terminal window.

## Broken links

If spidersuite finds broken links, the `not found count` value in the response is greater than 0. The response also includes `not found` information:

```
timeout count: 0
error count: 0
not found count: 5
not found: [
  {
    "url": "https://localhost:8443/brokenurl/",
    "linkedFrom": [
      "https://localhost:8443/"
    ]
  },
  ...
]
```

The `linkedFrom` information shows where spidersuite found this file.

## Broken redirects

If spidersuite finds a link that redirects to a missing page, the report contains the `redirectFrom` property, which lists the URLs for a chain of redirects. Any URL in that chain might be present in the content and contribute to the overall error. For example:

```
not found count: 1
not found: [
  {
    "url": "https://localhost:8443/brokenurlredirect2",
    "redirectFrom": [
      "https://localhost:8443/brokenurlredirect1",
      "https://localhost:8443/brokenurlredirect0"
    ]
  }
]
```

In this example, the original URL in the content is `https://localhost:8443/brokenurlredirect0`. This URL redirected several times and ended on `https://localhost:8443/brokenurlredirect2`, which returns the HTTP `404 Not Found` status code.

> **Important:** The `redirectFrom` property can have multiple heads to the redirect chain. For example, the list might show URLs A, B, C, and D, but A might redirect to B, C might redirect to D, and both B and D redirect to the offending URL. When spidersuite detects an error that results from a chain of redirects, more than one chain can redirect to the same faulty page so you must check the content for all parts of all chains specified by the `redirectFrom` list.

## Configure spidersuite

To meet your crawling and reporting needs, set one or more options in the spidersuite configuration file.

### Configuration file format

The spidersuite configuration file resembles the [eslint configuration file](http://eslint.org/docs/user-guide/configuring). 

Use the `extends` property to find either a referenced file or the default configuration if you specify `spider:default`. 

Supports only `.json` or `.js` files.

### Config file options

The following table outlines the full set of configuration options. For more details, see the `examples` directory.

For any pattern, spidersuite replaces `#{ROOT_URL}` with the extracted root URL, such as https://domain.example.com:5522). This lets you treat URLs that intentionally, or unintentionally, link to other hosts differently.

| Option | Description |
|:-------|:------------|
| `additionalPaths` | A list of paths to crawl, in addition to the initial URL provided. Use this option for hidden pages, which are pages that you cannot navigate from the main URL. |
| `excludePatterns` | A list of patterns that specify which URLs to not attempt. Default is spidersuite includes all urls it finds. |
| `includePatterns` | A list of patterns that specify which URLs to attempt. Default is spidersuite includes all URLs it finds. If specified, spidersuite will only fetch pages matching at least one of the patterns. |
| `titlePattern` | A regex pattern that indicates what the HTML title of the crawled pages on the same domain should contain. |
| `<error>WarnOnlyPatterns` | Reports the specified `<error>` as a warning rather than a failure. Value is either `hashNotFound` or `http<XXX>`, as follows:<ul><li>`hashNotFoundWarnOnlyPatterns`. Reports `404` errors as warnings.</li><li>`httpXXXWarnOnlyPatterns`. Reports the specified `XXX` errors as warnings. The `XXX` value is a number from 400 to 510.</li></ul> |
| `reportSpoolInterval` | A number that is greater than zero. Indicates the interval with which to report the current spool. The spool is the pages currently fetching. Useful for debugging. |
| `strictCiphers` | If `false`, the cipher list is relaxed. If `true`, a more strict version of ciphers is used over TLS. |
| `simplecrawlerConfig` | Spidersuite is based on [`simplecrawler`](https://www.npmjs.com/package/simplecrawler). This module has many configuration options. Use the `simplecrawlerConfig` option to set simplecrawler options. |
| `MAX_*_FROM` | By default, spidersuite reports only the first five broken links and redirects for a page. To change that default to report all broken links or redirects for each page, set the `MAX_LINKS_FROM` and `MAX_REDIRECTS_FROM` environment variables to `-1`.<blockquote><strong>Note:</strong> If the failure is in the header or footer and `MAX_LINKS_FROM` is set to `-1`, hundreds or thousands of entries appear in the `linkedFrom`section of the report, which makes the report hard to read.</blockquote> |
