# express-sitemap-xml [![travis][travis-image]][travis-url] [![npm][npm-image]][npm-url] [![downloads][downloads-image]][downloads-url] [![javascript style guide][standard-image]][standard-url]

[travis-image]: https://travis-ci.org/lydell/express-sitemap-xml.svg?branch=languages
[travis-url]: https://travis-ci.org/lydell/express-sitemap-xml
[npm-image]: https://img.shields.io/npm/v/@lydell/express-sitemap-xml.svg
[npm-url]: https://npmjs.org/package/@lydell/express-sitemap-xml
[downloads-image]: https://img.shields.io/npm/dm/@lydell/express-sitemap-xml.svg
[downloads-url]: https://npmjs.org/package/@lydell/express-sitemap-xml
[standard-image]: https://img.shields.io/badge/code_style-standard-brightgreen.svg
[standard-url]: https://standardjs.com

### Fork information

- Correct Content-Type header (application/xml).
- Support for [language links](#language-links).
- Updated dependencies.

### Express middleware to serve [`sitemap.xml`](https://en.wikipedia.org/wiki/Sitemaps) from a list of URLs

Create an Express middleware that serves `sitemap.xml` from a list of URLs.

This package automatically handles sitemaps with more than 50,000 URLs. In these
cases, multiple sitemap files will be generated along with a "sitemap index" to
comply with the [sitemap spec](https://www.sitemaps.org/protocol.html) and
requirements from search engines like Google.

If only one sitemap file is needed (i.e. there are less than 50,000 URLs) then
it is served directly at `/sitemap.xml`. Otherwise, a sitemap index is served at
`/sitemap.xml` and sitemaps at `/sitemap-0.xml`, `/sitemap-1.xml`, etc.

## Install

```
npm install @lydell/express-sitemap-xml
```

## Demo

You can see this package in action on [BitMidi](https://bitmidi.com), a site for
listening to your favorite MIDI files.

## Usage (with Express)

The easiest way to use this package is with the Express middleware.

```js
const express = require('express')
const expressSitemapXml = require('@lydell/express-sitemap-xml')

const app = express()

app.use(expressSitemapXml(getUrls, 'https://bitmidi.com'))

async function getUrls () {
  return await getUrlsFromDatabase()
}
```

Remember to add a `Sitemap` line to `robots.txt` like this:

```
Sitemap: https://bitmidi.com/sitemap.xml
```

## Usage (without Express)

The package can also be used without the Express middleware.

```js
const { buildSitemaps } = require('@lydell/express-sitemap-xml')

async function run () {
  const urls = ['/1', '/2', '/3']
  const sitemaps = await buildSitemaps(urls, 'https://bitmidi.com')

  console.log(Object.keys(sitemaps))
  // ['/sitemap.xml']

  console.log(sitemaps['/sitemap.xml'])
  // `<?xml version="1.0" encoding="utf-8"?>
  //  <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  //    <url>
  //      <loc>https://bitmidi.com/1</loc>
  //      <lastmod>${getTodayStr()}</lastmod>
  //    </url>
  //    <url>
  //      <loc>https://bitmidi.com/2</loc>
  //      <lastmod>${getTodayStr()}</lastmod>
  //    </url>
  //    <url>
  //      <loc>https://bitmidi.com/3</loc>
  //      <lastmod>${getTodayStr()}</lastmod>
  //    </url>
  //  </urlset>`
})
```

Remember to add a `Sitemap` line to `robots.txt` like this:

```
Sitemap: https://bitmidi.com/sitemap.xml
```

## API

### `middleware = expressSitemapXml(getUrls, base)`

Create a `sitemap.xml` middleware. Both arguments are required.

The `getUrls` argument specifies an async function that resolves to an array of
URLs to be included in the sitemap. Each URL in the array can either be an
absolute or relative URL string like `'/1'`, or an object specifying additional
options about the URL:

```js
{
  url: '/1',
  lastMod: new Date('2000-02-02'),
  changeFreq: 'weekly'
}
```

For more information about these options, see the [sitemap spec](https://www.sitemaps.org/protocol.html). Note that the `priority` option is not supported because [Google ignores it](https://twitter.com/methode/status/846796737750712320).

You can also pass each item as an _array_ to create [language
links](#language-links).

The `getUrls` function is called at most once per 24 hours. The resulting
sitemap(s) are cached to make repeated HTTP requests faster.

The `base` argument specifies the base URL to be used in case any URLs are
specified as relative URLs. The argument is also used if a sitemap index needs
to be generated and sitemap locations need to be specified, e.g.
`${base}/sitemap-0.xml` becomes `https://bitmidi.com/sitemap-0.xml`.

#### Language links

In order to [tell Google about localized versions of your
page](https://support.google.com/webmasters/answer/189077?hl=en#sitemap), use
arrays of language-url pairs:

```js
[
  {
    url: '/',
    lastMod: '2000-01-01',
    changeFreq: 'daily'
  },
  [
    {
      language: 'en',
      url: '/english/page.html'
    },
    {
      language: 'de',
      url: {
        url: '/deutsch/page.html',
        lastMod: new Date('2000-02-02'),
        changeFreq: 'weekly'
      }
    },
    {
      language: 'de-ch',
      url: {
        url: '/schweiz-deutsch/page.html',
        changeFreq: 'daily'
      }
    }
  ]
]
```

<details>

<summary>Result</summary>

```xml
<?xml version="1.0" encoding="utf-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://www.example.com/</loc>
    <lastmod>2000-01-01</lastmod>
    <changefreq>daily</changefreq>
  </url>
  <url>
    <loc>https://www.example.com/english/page.html</loc>
    <lastmod>${getTodayStr()}</lastmod>
    <xhtml:link rel="alternate" hreflang="en" href="https://www.example.com/english/page.html"/>
    <xhtml:link rel="alternate" hreflang="de" href="https://www.example.com/deutsch/page.html"/>
    <xhtml:link rel="alternate" hreflang="de-ch" href="https://www.example.com/schweiz-deutsch/page.html"/>
  </url>
  <url>
    <loc>https://www.example.com/deutsch/page.html</loc>
    <lastmod>2000-02-02</lastmod>
    <changefreq>weekly</changefreq>
    <xhtml:link rel="alternate" hreflang="en" href="https://www.example.com/english/page.html"/>
    <xhtml:link rel="alternate" hreflang="de" href="https://www.example.com/deutsch/page.html"/>
    <xhtml:link rel="alternate" hreflang="de-ch" href="https://www.example.com/schweiz-deutsch/page.html"/>
  </url>
  <url>
    <loc>https://www.example.com/schweiz-deutsch/page.html</loc>
    <lastmod>${getTodayStr()}</lastmod>
    <changefreq>daily</changefreq>
    <xhtml:link rel="alternate" hreflang="en" href="https://www.example.com/english/page.html"/>
    <xhtml:link rel="alternate" hreflang="de" href="https://www.example.com/deutsch/page.html"/>
    <xhtml:link rel="alternate" hreflang="de-ch" href="https://www.example.com/schweiz-deutsch/page.html"/>
  </url>
</urlset>
```

</details>

### `sitemaps = expressSitemapXml.buildSitemaps(urls, base)`

Create an object where the keys are sitemap URLs to be served by the server and
the values are strings of sitemap XML content. (This function does no caching.)

The `urls` argument is an array of URLs to be included in the sitemap. Each URL
in the array can either be an absolute or relative URL string like `'/1'`, or an
object specifying additional options about the URL. See above for more info
about the options.

The `base` argument is the same as above.

The return value is an object that looks like this:

```js
{
  '/sitemap.xml': '<?xml version="1.0" encoding="utf-8"?>...'
}
```

Or if multiple sitemaps are needed, then the return object looks like this:

```js
{
  '/sitemap.xml': '<?xml version="1.0" encoding="utf-8"?>...',
  '/sitemap-0.xml': '<?xml version="1.0" encoding="utf-8"?>...',
  '/sitemap-1.xml': '<?xml version="1.0" encoding="utf-8"?>...'
}
```

## License

MIT. Copyright (c) [Feross Aboukhadijeh](https://feross.org) and Simon Lydell.
