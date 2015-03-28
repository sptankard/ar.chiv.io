# ar.chiv.io
**A work in progress.** The design is being drafted right now. Watch the repository and [come shape the project](https://github.com/FiloSottile/ar.chiv.io/issues/2)! We'll use [the Issues section](https://github.com/FiloSottile/ar.chiv.io/issues/) as a mailing list.

**I'm looking for developers to join me!**

[Here](https://draftin.com/documents/385453?mode=presentation&token=vJDa78r0Ku2JsGeFHbw2LF-WEnkM1CntBHYa7QXnxxeA6joZ6KuUnzV7uKyls3s9paSgntlisg9ItFStbFTEST0) is a quick presentation on the project.

## Goals

Produce a **permanent**, *future-proof*, **faithful** archive of all the web pages you visit.

* Archived pages should be as static as possible, but they should include dynamic content as loaded in a browser. À la [archive.today](http://archive.today).  An archival copy of everything *exactly as you saw it*, plus some (that you didn't see).
* There should be broad support for plug-ins, both JS injected in pages and for the backend.
* Storage should be permanent, deduplicated and indexed.
* Full-text search on entire archive, with OS-specific UI integration.
* Browse your history even in offline mode!

## How

Here are just some ideas on how to build it. Have a better idea? [Share it](https://github.com/FiloSottile/ar.chiv.io/issues/2)!

* A local browser extension should store the URLs
* The browser-loading part should probably happen in [PhantomJS](http://phantomjs.org/) or [Selenium](http://docs.seleniumhq.org/)
* Then JS plugins should be injected.
* Then CSS should be embedded, forms neutralized, JS removed, links edited (?), images fetched and DOM snapshotted.
* Then the result should be stored encrypted and deduplicated Tarsnap style.

Suggestion: The local browser extension should handle not only storing the URLs, but also initiating and orchestrating the archiving events. This enables two things: avoid double-downloading; and storing the content *exactly* as visited by the user.

Suggestion: Use Selenium (not PhantomJS). Because: Tighter browser integration. This enables both store-exactly-as-visited, and easier integration of ar.chiv.io plugins (even with dynamic switching plugins on/off from the browser interface).

Suggestion: [Use the MAFF file format] (http://maf.mozdev.org/maff-file-format.html) for archival.  It has/includes several useful properties, such as:
* compressed (ZIP)
* internal links: "Enabling off-line navigation between pages in different archives"
* Metadata: original URL the page was saved from; date/time of the save operation; "arbitrary extended metadata", such as "browser's scroll position in the page, text zoom level, and more"

The [MAF browser extension] (http://maf.mozdev.org/) already implements the grunt work.

TODO: work out a format for diff'ing the MAFF files, for versioning. (Just diff each internal file?)

TODO: structural anatomy of the data store

### Program structure

* A set of browser-specific extensions (Firefox, Chrome, ...)
* ar.chiv.io core daemon, handling deduplication, storage, etc.
* ar.chiv.io CLI tools, to manually interact with store if desired
  * purge entries by site, export/import/partition stores (per browser, date, etc.)
  * perform on-demand web crawls/whole-site-fetching
* A set of index-service integration plugins, exposing the data for indexing by specialized services (Tracker, ...)
* Plugins (for JS-injection etc.), interoperable between browser extension and web-crawl CLI tool

### Step-by-step

1. User loads page
2. Browser extension makes a MAF snapshot (executing any before/after plugin-events via Selenium, if applicable)
3. User interacts with page
4. Browser extension detects changes and makes further MAFF diff-snapshots when necessary
5. [Repeat steps 3-4 indefinitely]
6. User closes page
7. Browser extension makes a final snapshot, in background (executing plugins, if applicable)

### Browser extensions

* Hooks and config for automatic snapshotting
* Plugins for JS-injection, etc.
* Play nice with other browser extensions (AdblockPlus, Greasemonkey...)

### Privacy

* Consider NOT encrypting local storage: provides a false sense of security (unless user wishes to enter password every time they access their archive).  Of course, any off-site storage/backup must be encrypted.
* Obey Private Browsing Mode -- do not archive/index.
* Normal Firefox "Forget this site" feature → should trigger deletion from archivio store?

### Session-internal versioned history

Suggestion: session-internal versioned-history (conceptual parallel to the versioned history that applies to a URL whose content changes as it is visited repeatedly over weeks/months).

Why? A problem arises from the desire to save both:
* The page exactly as the user saw it
* The full content of the page, not just parts (scroll-loaded content, etc.)

An additional issue arises: avoiding double-downloading (which wastes bandwidth, and alerts servers to something fishy).

Both these issues can be solved via a versioned-history approach, storing successive versions of the page *as it is visited*. The first layer is the page as the user sees it upon page load, and subsequent content changes can be saved as diff layers.  Upon page close, a Selenium event can expand any remaining content (comments, video downloads, etc. -- as per installed plugins).

TODO: Define an heuristic for when to snapshot (how much of a change counts).

### Indexing

Full-text search on your browsing history.  A user-wide store that integrates browsing from multiple sources (browsers); but that records the origin (Firefox- and Chrome-visited pages are not conflated).

Integrate with desktop indexing services.

* [MetaTracker](https://wiki.gnome.org/Projects/Tracker)
* [YaCy](http://www.yacy.net/en/)
* Others...?

Both MetaTracker and YaCy support indexing HTML files.  Would indexing [Readability.js](https://github.com/mozilla/readability) output provide any better results?

### Deduplication
What backend to use for the deduplication? How to calculate diffs on MAFF files?
## Plug-in ideas

* Auto scroll to load more content
* Expand comments (Reddit, ...)
* Dismiss pop-ups (Quora, ...)
* youtube-dl for supported sites
* Blacklist sites - regexes
* Fetch whole site (Readthedocs, ...)
* Repo cloning (GitHub, ...)
* Run Readability and store/index the result
