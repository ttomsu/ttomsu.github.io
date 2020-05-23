---
title: The Morgan Housel Archive
date: 2020-05-23
tags: morgan_housel
---

Back in 2010, I expanded my interest in personal finance out into investments. My grandfather had recently passed away, and left me with a small chunk of change that my grandmother insisted I keep invested in the stock market. I didn't know the first thing about investing.

I stumbled onto the [Motley Fool](https://www.fool.com) -- a news and commentary site about all things stock market. Their homepage frequently featured a writer I looked forward to reading named [Morgan Housel](http://twitter.com/morganhousel).

Morgan's focus was not so much on the day-to-day, quarter-to-quarter stuff. Instead, he focused on timeless areas where investing meets psychology, behavior, rationality, and expectations.

In 2016, he moved on to the [Collaborate Fund](https://www.collaborativefund.com/), a venture capital firm focused on funding companies at the [intersection](https://www.collaborativefund.com/about/) of "better for me" and "better for the world." I continued to follow Morgan's writing here, where he's been cranking out home run articles ever since.

To thank Morgan for 10 years of influential, thought provoking, and impactful pieces, I'd like to introduce a way to archive Morgan's writing (_all_ of it) for yourself. I should have been bookmarking his "best of" throughout the years, but like investing, the second-best time to start is now.

Here are the URL lists of all of his work from the Motley Fool and Collaborative Fund. Details on how I scraped these together and what you can do with them are below.

* [All articles](/assets/misc/posts/morganhousel/all.txt)
  * (2016-present) [Collaborative Fund](/assets/misc/posts/morganhousel/cf.txt) (up through 2020-05-22)
    * [Long form PDFs](/assets/misc/posts/morganhousel/cf-pdfs.txt)
  * (2008-2016) [The Motley Fool](/assets/misc/posts/morganhousel/mf-all.txt)
    * [`/investing/general`](/assets/misc/posts/morganhousel/mf-investing-general.txt) - where most of the "timeless" stuff is.


## Generating the URL lists

Luckily, both MF and CF have pages devoted to all things written by specific authors:

* [http://fool.com/author/1611/](http://fool.com/author/1611/)
* [https://www.collaborativefund.com/blog/authors/morgan/](https://www.collaborativefund.com/blog/authors/morgan/)


### Fool.com

Fool.com structures their author pages as a pseudo-infinite scroll. Watching the URL bar shows a variation of `https://www.fool.com/author/1611/?page=2` as new pages are loaded.

After some trial and error, the entire archive goes up to _134_ pages. A simple script can be used to fetch them all:

```shell
$ for i in {1..134}; do
  wget https://fool.com/author/1611/?page=$i && sleep 1;
done
```
Inspecting the pages reveals:

```html
<a href="/investing/general/2016/03/30/things-im-pretty-sure-about.aspx" title="Things I'm Pretty Sure About " data-id="article-list">
  <article>
    <div class="text">
      <h4>Things I'm Pretty Sure About </h4>
      <div class="story-date-author">Morgan Housel | Mar 30, 2016</div>
      <p class="article-promo">We're all in the same boat.</p>
    </div>
  </article>
</a>
```

The `data-id="article-list"` is a nice indicator pull out the line that has all of the links. Here, we'll grab all of lines from the raw HTML, then remove the cruft before and after the `href` link:

```shell
$ grep 'data-id\=\"article-list\"' --no-filename index.html\?page\=* > raw-links
$ sed 's/^.*href\=\"/https\:\/\/www.fool.com/' raw-links | sed 's/\.aspx".*$/\.aspx/' | sort > mf-all.txt
```

Sample Output:
```
https://www.fool.com/investing/2016/05/24/how-to-fail-well.aspx
https://www.fool.com/investing/2016/05/24/how-to-make-better-investment-decisions.aspx
https://www.fool.com/investing/2016/05/26/why-everyone-disagrees-about-the-economy.aspx
https://www.fool.com/investing/2016/06/02/pithy-wisdom-that-changed-how-i-think-about-invest.aspx
https://www.fool.com/investing/2016/06/07/explaining-investing-in-ways-that-make-sense.aspx
https://www.fool.com/investing/2016/06/09/investing-the-market-and-beyond-a-day-with-foolish.aspx
https://www.fool.com/investing/2016/06/13/two-trends-were-not-excited-enough-about.aspx
https://www.fool.com/investing/2016/06/14/sherlock-holmes-on-how-to-be-a-better-investor.aspx
https://www.fool.com/investing/2016/06/16/what-i-learned-buying-a-house.aspx
...
```

### Collaborativefund.com

Collaborativefund.com makes things a bit easier (no fancy paging), but it also appears the data is loaded asynchronously. So this simple `wget` doesn't have any of the links:

```shell
$ wget https://www.collaborativefund.com/blog/authors/morgan/
```

A simple workaround is to just right-click anywhere on the page in a browser and "Save as..." --> "Website, Complete" --> Save.

Inspecting the page, one can find repeated elements like this:

```html
<li class="post-item"><h4><a href="https://www.collaborativefund.com/blog/the-shock-cycle/">The Shock Cycle</a></h4>
  <small class="post__meta"><time class="faded">Apr 2, 2020</time>
    <span class="faded">by</span>
    <span class="js-author" data-author="Morgan Housel">
      <a class="faded faded--60 js-author__title js-related__author" href="https://www.collaborativefund.com/blog/authors/morgan/">Morgan Housel</a>
    </span>
  </small>
</li>
```

From here, we can `grep` for the links in the same way, looking this time for `<li class='post-item'>` elements:

```shell
$ grep 'post-item' Morgan\ Housel\ ·\ Collaborative\ Fund.html > raw-links
$ sed 's/^.*href\=\"//' raw-links | sed 's/">.*$//' | sort > cf-all.txt
```

Sample output:

```
https://www.collaborativefund.com/blog/100-little-ideas/
https://www.collaborativefund.com/blog/2020-what-a-time-to-be-alive/
https://www.collaborativefund.com/blog/23-books-that-changed-my-life/
https://www.collaborativefund.com/blog/acceptable-flaws/
https://www.collaborativefund.com/blog/a-chat-with-daniel-kahneman/
https://www.collaborativefund.com/blog/a-decade-watching-the-craziest-game/
https://www.collaborativefund.com/blog/a-few-thoughts-on-public-speaking/
...
```

## Archiving The Pages

It took me several iterations, but I think the following command should work for each of the text files full of links you just created:

```shell
$ INPUT_FILE=cf-all.txt
$ wget \
  --input-file $INPUT_FILE \
  --page-requisites \
  --span-hosts \
  --domains www.collaborativefund.com,www.fool.com,g.foolcdn.com \
  --convert-links \
  --wait=1.5 \
  --random-wait \
  --adjust-extension \
  --output-file=wget.log \
  --background
```

Lets go through each of these options, because each one is kind of important:

* `--input-file` allows us to specify a file with the list of URLs to fetch
* `--page-requisites` fetches the CSS, JavaScript, and images that make the page look nice and work well. Technically, we'd really only like the CSS and images, but I didn't find a way to get just those. By default, it only fetches them from the same host domain, so we use:
  * `--span-hosts` gets required resources from other host domains, and
  * `--domains` specifies to only fetch resources from these whitelisted domains. It appears MF uses `g.foolcdn.com` as their host for images, JS, and CSS.
* `--convert-links` changes all reference to images/CSS/JS to use the locally fetched ones (thanks `--page-requisites`!)
* `--wait` puts some time between requests so we don't _hammer_ the site
* `--random-wait` randomizes the wait between 0.5x and 1.5x of the `--wait` time
* `--adjust-extension` renames `.aspx` files to `.html`
* `--output-file` to capture the logs of each request
* `--background` to put it in the background. It'll take a while.

In the end you'll have something like this:

```shell
$ tree www.collaborativefund.com/
www.collaborativefund.com/
├── assets
│   ├── css
│   │   └── all.css
│   ├── fonts
│   │   ├── ss-collaborative-fund-social-circle.eot
│   │   ├── ss-collaborative-fund-social-circle.eot?
│   │   ├── ss-collaborative-fund-social-circle.svg
│   │   ├── ss-collaborative-fund-social-circle.ttf
│   │   ├── ss-collaborative-fund-social-circle.woff
│   │   ├── ss-collaborative-fund-social-regular.eot
│   │   ├── ss-collaborative-fund-social-regular.eot?
│   │   ├── ss-collaborative-fund-social-regular.svg
│   │   ├── ss-collaborative-fund-social-regular.ttf
│   │   └── ss-collaborative-fund-social-regular.woff
│   ├── images
│   │   ├── home--cities--overhang.svg
│   │   ├── home--cities.svg
│   │   ├── home--consumer--small.svg
│   │   ├── home--consumer.svg
│   │   ├── home--consumer--text.svg
│   │   ├── home--health--overhang.svg
│   │   ├── home--health--small.svg
│   │   ├── home--health.svg
│   │   ├── home--kids--small.svg
│   │   ├── home--kids.svg
│   │   ├── home--kids--text.svg
│   │   ├── home--money--overhang.svg
│   │   ├── home--money.svg
│   │   └── home--money--text.svg
│   └── js
│       └── all.js
├── blog
│   ├── 100-little-ideas
│   │   └── index.html
│   ├── 2020-what-a-time-to-be-alive
│   │   └── index.html
│   ├── 23-books-that-changed-my-life
│   │   └── index.html
│   ├── acceptable-flaws
│   │   └── index.html
    ... snip ...
│   ├── you-can-see-where-this-is-going
│   │   └── index.html
│   ├── you-have-to-live-it-to-believe-it
│   │   └── index.html
│   └── you-played-yourself
│       └── index.html
└── uploads
    ├── 0labr_spnd0-brad-fickeisen.jpg
    ├── 10-5591a8.png
    ├── 11-5dd870.png
    ... snip ...
    ├── vladimir-chuchadeev-86192.jpg
    ├── vqpqjitnqvw-anton-belashov.jpg
    ├── wesley-tingey-mvLyHPRGLCs-unsplash.jpg
    ├── wim-arys-523715-unsplash.jpg
    ├── yb0qs65azmc-anders-jilden-2b4e51.jpg
    └── yuriy-kleymenov-1102408-unsplash.jpg

227 directories, 596 files
```


## Long Form PDFs

Morgan has published numerous long form PDFs, which I'd also like to archive. Now that we have all of his posts, we can search for `.pdf` references and grab those too:

```shell
$ grep --no-filename "collaborativefund.com/uploads/.*\.pdf" -r www.collaborativefund.com/blog/ > raw-pdf-links
$ sed 's/^.*href\=\"//' raw-pdf-links | sed 's/pdf">.*$/pdf/' | grep "pdf" | sort | uniq > cf-all-pdfs.txt
```

From here, you can use a simple `wget --input-file cf-all-pdfs.txt --wait=2` to pull all of the PDFs and put them in the `uploads` folder.


## Viewing the Files

Launch any of these pages with your browser:

```shell
$ google-chrome google-chrome ./www.collaborativefund.com/blog/true-at-once/index.html
```

You should notice that this is served from your local machine.

---

Thank you, Morgan!
