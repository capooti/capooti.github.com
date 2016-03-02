---
categories: GIS, Python, blogofile
date: 2011/03/09 19:59:10
guid: http://www.paolocorti.net/why-I-have-moved-my-blog-to-blogofile
permalink: http://www.paolocorti.net/2011/03/09/why-I-have-moved-my-blog-to-blogofile
tags: GIS, Python, blogofile
title: Why I have moved my blog to Blogofile
---
So, after five years of blogging I have sent <a href='http://oldblog.paolocorti.net/'>my Wordpress blog to pension</a> and I have moved my blog to Blogofile.

You may wonder why I have moved my blog from a spread and fully featured platform to an almost unknown exotic one.
Basically I wanted to have the following advantages:

* Python framework, with the possibility to write in this language plugins and extensions
* ability to write my blog posts offline, in a markup format like reST or markdown, with vi
* blog system able to generate static content from the posts written in the markup language, in a Sphinx fashion, without the overhead of a database and a server language. Note that static also means: highly reliable, secure and inexpensive hosting
* possibility to commit the posts to git, and automatically update the blog site after push

Thanks to Matthews Dougal, I have found out that Blogofile, by Ryan McGuire, is an awesome answer to all this request, and I have now became a proudly early adopter of this blog engine.

I am actually using the latest development version from gitHub, and I could easily import all of my blog posts, categories and tags from Wordpress via a Python script included in the source (there is a script also to import from Blogger).
I then converted all posts to markdown (I have found out that it was the easier way to go from html to a markup format, without messing too much the html Wordpress post text).
I did not want to loose the possibility to have comments at my posts, so I have opted to use the excellent Disqus service, that let you import your comments from the main blog platforms as well: Blogofile seamless integrates with Disqus comments in your posts.

The only thing that Blogofile could not offer me at this time was the ability to automatically creates links for certain keywords at my post.
I did not want to loose this feature (from <a href='http://wordpress.org/extend/plugins/seo-automatic-links/'>SEO Smart Links plugin in Wordpress</a>), and it was quiete easy to write a <a href='https://github.com/EnigmaCurry/blogofile/pull/85'>Python filter</a> that is doing the trick for me.

Finally I created a hook in my git repository at my host, this way every time a pull is performed in git, the html of the blog is rebuilt and is live at the website: brilliant!










