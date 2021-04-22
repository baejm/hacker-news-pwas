<p align="center">
  <a href="https://hnpwa.com/">
    <img src='https://raw.githubusercontent.com/tastejs/hacker-news-pwas/b3f3d40b9e4bd385dbb973d238ce207aed1f60eb/media/logo.png' width='400px'/>
  </a>
</p>

<p align="center">
  Hacker News readers as  <a href="https://g.co/ProgressiveWebApps">Progressive Web Apps</a>. A spiritual successor to <a href="https://github.com/tastejs/todomvc">TodoMVC</a>.
</p>

<p align="center">
  <a href="https://travis-ci.org/tastejs/hacker-news-pwas"><img alt="Build Status" src="https://travis-ci.org/tastejs/hacker-news-pwas.svg?branch=master"></a>
</p>

---

## Implementations

See our [site](https://hnpwa.com/) or the `site/apps` directory for the current
list of implementations. 

## Specification

Each implementation must include:

* Views: Hacker News Top Stories, New, Show, Ask, Jobs & threaded Comments
  * Each of these should use routing to enable sharability. For reference, see the [PreactHN](https://hn.kristoferbaxter.com/) implementation.
* App must display 30 items per-page for story list views
* App must be a [Progressive Web App](https://g.co/ProgressiveWebApps)
* App must score over a 90/100 using [Lighthouse](https://github.com/GoogleChrome/lighthouse)
* App must aim to be interactive in under 5 seconds on a Moto G4 over 3G. Use [WebPageTest](https://www.webpagetest.org/easy) using the auto-selected Moto G4 + Faster 3G setting to validate "Time to interactive"
  * We look at numeric Lighthouse scores for TTI as well as a manual inspection of the application's Timeline "trace" and [Filmstrip](https://www.webpagetest.org/video/compare.php?tests=170514_00_bb389f33405b558ea644b37f565c8a56-r:1-c:0) as a sanity check.
* App must use the [Application Shell](https://developers.google.com/web/fundamentals/architecture/app-shell) pattern to instantly load the skeleton of the UI on repeat visits
* App is responsive on desktop and mobile, making best use of available screen real-estate. See [Vue HN](https://vue-hn.now.sh/top) as an example.
* App must do its best to work cross-browser

Optionally:

* App supports offline caching of HN data (e.g similar to the 'Offline Mode' in ReactHN)
* App may use server-side rendering so displaying content is resilient to JS not loading on the network

User interface:

* At this time, HNPWA does not prescribe a specific stylesheet or theme for implementations. We will be aiming to provide this in the near future similar to how we do with TodoMVC.

### Data sources

* [Official real-time Hacker News API powered by Firebase](https://github.com/HackerNews/API)
* [Unofficial Hacker News API](https://github.com/cheeaun/node-hnapi) by cheeaun

If using the Firebase powered API please use 30 stories per-page to ensure consistency between implementations using the Unofficial API as well as the actual [Hacker News website](https://news.ycombinator.com/)

## Network settings

* Emerging Markets: Chrome Beta on a Motorola G (gen 4) tested from Dulles, Virginia on a 400 Kbps 3G connection with 400ms of latency. Tested with [WebPageTest](https://www.webpagetest.org/easy) using the auto-selected Moto G4 + Emerging Markets setting.
* Faster 3G: Chrome Beta on a Motorola G (gen 4) tested from Dulles, Virginia on a 1.6 Mbps 3G connection with 300ms of latency. Tested with [WebPageTest](https://www.webpagetest.org/easy) using the auto-selected Moto G4 + Faster 3G setting.
* `Time to Interactive` readings taken from linked Lighthouse results in WebPageTest.

## License

Each implementation preserves the license noted in the linked to applications.





# HNPWA API

**A fast, CDN delivered, aggregated Hacker News API**

https://api.hnpwa.com/v0/news/1.json

## Why?

Everything in the official Hacker News API is either 
[an item](https://hacker-news.firebaseio.com/v0/item/8863.json?print=pretty) or 
[a list of items](https://hacker-news.firebaseio.com/v0/topstories.json?print=pretty).

This structure is not ideal for building fast loading Progressive Web Apps since it will require multiple network requests
to get the needed payload for page render. 

The HNPWA solves this problem by aggregating each item in the id list into item feeds.

## Item feeds

There are five item feeds.

| Name | URL |
| --- | --- |
| News| https://api.hnpwa.com/v0/news/1.json |
| Newest | https://api.hnpwa.com/v0/newest/1.json |
| Ask | https://api.hnpwa.com/v0/ask/1.json |
| Show | https://api.hnpwa.com/v0/show/1.json |
| Jobs | https://api.hnpwa.com/v0/jobs/1.json |

### Schema

Each item feed returns an array of `FeedItem`.

```ts
export interface FeedItem {
  id: number;
  title: string;
  points?: number | null;
  user?: string | null;
  time: number;
  time_ago: string;
  comments_count: number;
  type: string;
  url?: string;
  domain?: string;
}
```

### Paging 

Item feeds can be paged by accessing the next index in the page. Each page starts at 1 and each feed has a different ending page.

[https://api.hnpwa.com/v0/news.json/2.json](https://api.hnpwa.com/v0/news.json/2.json)
  
| Name | Max Pages |
| --- | --- |
| News| 10 |
| Newest | 12 |
| Ask | 2 |
| Show | 2 |
| Jobs | 1 |

## Individual items

Feeds provide the top level view of an item, but other details like comment threads are available at the individual item level.

[https://api.hnpwa.com/v0/item/13831370.json](https://api.hnpwa.com/v0/item/13831370.json)

### Schema 

```ts
export interface Item {
  id: number;
  title: string;
  points: number | null;
  user: string | null;
  time: number;
  time_ago: string;
  content: string;
  deleted?: boolean;
  dead?: boolean;
  type: string;
  url?: string;
  domain?: string;
  comments: Item[]; // Comments are items too
  level: number;
  comments_count: number;
}
```

## Users

Users are retrieved by username.

[https://api.hnpwa.com/v0/user/davideast.json](https://api.hnpwa.com/v0/user/davideast.json)

### Schema

```ts
export interface User {
  about?: string;
  created_time: number;
  created: string;
  id: string;
  karma: number;  
}
```

## Local Development

The HNPWA API uses the [hnpwa-api](https://github.com/davideast/hnpwa-api/) module and runs on [Cloud Functions](https://firebase.google.com/docs/functions/) and [Firebase Hosting](https://firebase.google.com/docs/hosting/functions).
If you want to run the API while offline you can globally install the module to serve offline.

```bash
npm i -g hnpwa-api
hnpwa-api --save # saves current HN API data set offline
hnpwa-api --serve --offline --port=4000
```

