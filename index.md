---
layout: default
---

<div class="card">
{%- capture text -%}

# The blogging craze had me too

Hello, Vincenzo **Scalzi** here!

What if we could do better? How can we learn from our collective mistakes? It is high time to explore the blogging
universe and to contribute my own ideas into the mix. This blog comes in addition to my [Twitter feed](https://twitter.com/vcz_fr){:rel="nofollow"}
which contains my curated selection of news and ideas to explore.

Where does this wish to blog came from? Like anyone else, I have ideas. I wish to explore and share mine in the hopes to
produce the spark that will lead you to greater ideas. Similarly, I wish for you to help your community so that our
paths may cross one day.

As for this site, it is open source, hosted and delivered by [Cloudflare](https://www.cloudflare.com/){:rel="nofollow"}
and uses [Jekyll](https://jekyllrb.com/){:rel="nofollow"} to generate content.

There is no server to manage, no hosting costs or headaches, just a domain. Just a domain with free content you will
appreciate. I can only hope you will like it as I will never add any third party comment tool or analytics. If you do
enjoy the contents, come by [the place that hosts it](https://github.com/vcz-fr/blog){:rel="nofollow"} and leave your
mark. Or check out my [personal website](https://vcz.fr), my [Meetup notes](https://meetups.vcz.fr) or my [apps](https://apps.vcz.fr).

Do not forget to [leave some feedback](https://apps.vcz.fr/app/feedback/?appid=DW7RbJ8FWHm5) so that I can make your
experience better.

Have a nice visit!

{%- endcapture -%}
{{ text | markdownify }}
</div>

<div class="card preview">
{%- capture text -%}
## Latest post: [{{ site.posts.first.title }}]({{ site.posts.first.url }})

Posted {{ site.posts.first.date | date: "%A %B %d, %Y" }}

> {{site.posts.first.excerpt | strip_html | truncatewords: 100}}

_[Read more]({{ site.posts.first.url }})&hellip;_
{%- endcapture -%}
{{ text | markdownify }}
</div>