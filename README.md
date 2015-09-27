#vRatpack Posting Guidelines

[![Join the chat at https://gitter.im/vratpack/vratpack.github.io](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/vratpack/vratpack.github.io?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
This is a basic feature overview on vRatpack to get you familiar with posting.

---

##Jekyll
This blog is using the static-site generator [Jekyll](http://jekyllrb.com) using the [Liquid](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) template engine. This means that content, usually provided in a [Markdown](http://en.wikipedia.org/wiki/Markdown) file (.md) is processed into a static webpage (.html).
We took care about all the setup required for you, so all you really have to do is get familiar with Markdown by reading this Readme and start posting. Note, that even this Readme was written using Markdown, you can [view it in raw here](https://github.com/vratpack/vratpack.github.io/blob/master/README).

All posts have to follow a couple of conventions:
- place your posts into the [_posts](https://github.com/vratpack/vratpack.github.io/tree/master/_posts) folder
- name them using the syntax `YYYY-MM-DD-your-post-title-seperated-by-hyphens`
- make sure you use the file-extention `.md` for your posts
- use [Front Matter](http://jekyllrb.com/docs/frontmatter/) and [Markdown](http://en.wikipedia.org/wiki/Markdown) as explained in this readme

In addition you should get started with [Git](http://git-scm.com/) which we use for version control. There are a lot of [quick start guides](https://gist.github.com/blakerohde/6138015) out there.
Of course, if you want you can also use the [GitHub web interface](https://github.com/vratpack/vratpack.github.io/tree/master/_posts) and write your posts stright from your webbrowser if you like to.

Once a new post is created GitHub - to be more precise [GitHub-Pages](https://pages.github.com/) - will start a new Jekyll build job and compile your post, which should then soon appear at [www.vratpack.com](https://www.vratpack.com).

---

##Front Matter
A typical [Front Matter](http://jekyllrb.com/docs/frontmatter/) should be included in every post and look as follows.

```
---
layout: post
title: "Title Of This Post"
date: 2015-03-20 21:59:15
updated: 2015-03-20 22:30:15
comments: true
categories:
tags: [testtag-1, testtag-2]
author:
    name: "Author Display Name"
    twitter: "TwitterAccountName"
description: "Description Of This Post"
excerpt_separator: <!--more-->
---
```

The Front Matter controls how your post should look like and what features it should include. Note that some variables are optional.

###Title
Is the title of your post. You should use the same title within the syntax of the Markdown filename, as described above.

###Date
Is the date when you released your post.* Please be aware of the proper formatting, especially when it comes to date-time.*

###Comments
Controls if comments should be allowed for this post. If set true, [Disqus](https://disqus.com) will be enabled in your post.
If false or empty, no comments will be used.


![comments](/assets/site/readme/comments.png)

###Tags
One or more tags (comma seperated) to include in your post.
If tagged, the tags used will be displayed on the bottom of your post. E.g.: if you'd tag your post using the tags:

- tag1
- tag2

at the bottom of your post tags would be displayed like this:

![tags](/assets/site/readme/tags.png)

and your post would be listed on the tags pages e.g.
- `https://vratpack.com/blog/tag/testtag-1/`
- `https://vratpack.com/blog/tag/testtag-2/`

As you may have noticed, the url path to the tag is not the same as the tag display name. This is controlled within the [tags data file](https://github.com/vratpack/vratpack.github.io/blob/master/_data/tags.yml).
In fact, if you want to create a new tag that was never used before, you **first have to create it** within the [tags data file](https://github.com/vratpack/vratpack.github.io/blob/master/_data/tags.yml).
This is due to a limitation of Github-Pages, which doesn't allow using custom Jekyll plug-ins. The syntax for tag creation in the tags data file is:

```
- slug: tag-url-path-name
  name: tag-display-name
```

In addition you'll have to create a new Markdown file in the [tag folder](https://github.com/vratpack/vratpack.github.io/tree/master/tag). It has to be named `tag-url-path-name.md` and has to be formatted like this:

```
---
layout: tags
tag: tag-url-path-name
permalink: blog/tag/tag-url-path-name/
---
```

Only if all those prerequisite are met your new tag will work as expected.

###Author.Name
The name of the author of this post. Will be displayed below the post title if set.

###Author.Twitter
The twitter account of the author of this post. Will be linked with the name of the author if set. Do use the Twitter account name here only, *not* the full URL to your profile.

---

##Markdown Processor
We're using the [redcarpet markdown processor](https//github.com/vmg/redcarpet) in [jekyll](http://jekyllrb.com/). Redcarpet is a feature rich markdown processor. This section will give you some information on how it works and what's possible using it.
First of we can control which features - called extentions - we want using our config.yml. A list of valid extentions for Redcarpet can be [found here](https://github.com/vmg/redcarpet/blob/v2.2.2/README.markdown#and-its-like-really-simple-to-use).
Currently we use the following extentions:

```
redcarpet:
  extensions: ["no_intra_emphasis", "fenced_code_blocks", "autolink", "strikethrough", "superscript", "footnotes", "highligh", "quote", "underline", "tables"]
```

Make sure you’re looking at the README for the right version of Redcarpet: Jekyll currently uses v2.2.x, and extensions like footnotes and highlight weren’t added until after version 3.0.0.

---

#Highlight Support
We got highlight support for this languages:

- markup
- css
- javascript
- java
- http
- c
- go
- swift
- objectivec
- ini
- git
- powershell
- markdown
- ruby

At this moment we're using the built-in [pygments highlighter](http://pygments.org/) and the [redcarpet markdown processor](https//github.com/vmg/redcarpet).
Because of the way how redcarpet works and limitations on using custom plugin with github-pages, we have two possible highlight styles for code fencing.

The first posting style is using HTML `{% highlight %}` liquid-tag, e.g.:

```
{% highlight javascript linenos %}
	var now = new Date();
	var days = new Array('Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday');
	var months = new Array('January','February','March','April','May','June','July','August','September','October','November','December');
	var date = ((now.getDate()<10) ? "0" : "")+ now.getDate();
	function fourdigits(number)
	{
		return (number < 1000) ? number + 1900 : number;
	}
	today =  days[now.getDay()] + ", " +
	         months[now.getMonth()] + " " +
	         date + ", " +
	         (fourdigits(now.getYear())) ;
	document.write(today);
{% endhighlight %}
```

This will result in a code-block highlighted for the specified language including line-numbers, e.g.:

![codeblock-complex](/assets/site/readme/codeblock-complex.png)

The second posting style is using [GitHub flavored Markdown](http://doc.gitlab.com/ce/markdown/markdown.html) by simply using three back-ticks ` ``` `. After the back-tick you have to specify the language to use for highlighting.
Redcarpet will convert this into a HTML `code` element and set the class of that element to `class="languagename"` which is used by many highlighters - including pygments - for the syntax highlighting.

    ```javascript
	today =  days[now.getDay()] + ", " +
	months[now.getMonth()] + " " +
	date + ", " +
	(fourdigits(now.getYear())) ;

    ```

This style will result in a code-block highlighted for the specified language, e.g.:

![codeblock-complex](/assets/site/readme/codeblock-simple.png)

---

##Markdown Support
Basically Redcarpet supports [GitHub flavored Markdown](http://doc.gitlab.com/ce/markdown/markdown.html) which is Markdown with extras.
You can read all about the supported Markdown [here](https://help.github.com/articles/markdown-basics/) and [here](https://guides.github.com/features/mastering-markdown/), [here](https://help.github.com/articles/github-flavored-markdown/)and of course [here](http://doc.gitlab.com/ce/markdown/markdown.html).

The following will give you some examples on

###Inline Code
Redcarpet also has support for GitHub style inline code. Use a single back-ticks around code to get it formatted, e.g.:

```
Lorem ipsum dolor `var i = 5;` sit amet, consetetur sadipscing elitr, sed diam nonumy
```

will result in

![inline code](/assets/site/readme/inlinecode.png)

###Footnotes
```
Sentences with a footnotes.[^1]

[^1]: This is a footnote.
```

will result in

![footnotes](/assets/site/readme/footnotes.png)

###Tables

```
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```

will result in

![tables](/assets/site/readme/tables.png)

####Gist Integration
Using gist for code snippets? Great, we got support for that too, simply use

```
{% gist 7597812 %}
```

will result in

![gist](/assets/site/readme/gist.png)

###Images
If you want to post images, create a subfolder within the `/assets/posts/` folder and puts your images in there, e.g.: `/assets/posts/YYYY-MM-DD-your-post-title-seperated-by-hyphens/image.png`. You shouldn't use full qualified URL paths to the image because if we ever change domains, the links will be broken. However: if for whatever reason you choose to include content using the full URL path, e.g. using external images, make sure to use *https://* instead of *http://* because otherwise the browser will display certificate warnings to the user when loading the non-secure content over a secure connection.
The syntax for including images is default Markdown:

```
![Imagename](/assets/posts/YYYY-MM-DD-your-post-title-seperated-by-hyphens/image.png)
```

The image will auto-scale to fit the pages width, which is fixed to 720 pixel. If you want to use caption on your figures, you may use inline HTML to do so, e.g.:

    <figure>
	  <img src="https://octodex.github.com/images/yaktocat.png">
      <figcaption>Fig 1. the honorable Yaktocat. [GitHub](https://www.github.com))</figcaption>
    </figure>

will result in:

![yaktocat](/assets/site/readme/yaktocat.png)


###Excerpt
By default Jekyll will take the first paragraph as an excerpt of the post for the post index page. Using the **excerpt_separator** Front Matter you can put a little more control what part of your post to use as excerpt. E.g. if you use **excerpt_separator: <!--more-->** you can then just start writing your post and anywhere in your post write the <!--more--> tag to seperate excerpt from the rest of your post. The text before the excerpt separator will be used as excerpt.


That's it for now. As features may follow, thus guide will be updated.
