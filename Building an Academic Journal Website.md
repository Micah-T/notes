
# Introduction and Requirements
I recently built the website for the [Westmarch Literary Journal](https://westmarchjournal.org/) at Patrick Henry College. This is an academic journal website, so there are some requirements which are not commonly found in other websites:
1. [Maintainability and Readibility](#maintainability-and-readibility).
2. [Citation tools](citation-generator)
3. [Inclusion for Google Scholar and similar services](inclusion-for-google-scholar)

# Maintainability and Readibility
Academic journal websites must be readible and maintainable for a long period of time. Academic study seeks to increase the sum of knowledge that is available while also building certainty in this knowledge. To increase the sum of available knowledge requires readibility--our work is not particularly useful if it is inaccessible. To make the knowledge more certain requires maintaining this access--if cited works can no longer be found, this makes verifying research impossible. 

Readibility requires making websites easily accessible. This is a complex and frequently dicussed topic, though it can be simply summed: one ought to avoid doing anything which makes a website hard to read; that is, websites should be no more complex for the user's computer than is necessary. Thus, a correctly-authored HTML with few scripts is how an academic website should be authored. 

Maintainability requires that the content be easily updated and maintained for a long time. The website author must consider what format the content is stored in--if the system which processes the content becomes unavailable, the content must be easily converted to another format. I chose to use Markdown to store the content for this reason. [Pandoc](https://pandoc.org/) allows for easy conversion to and from Markdown, and Markdown itself is easily readible in a text editor. In addition, many other web publishing tools natively support Markdown. I also chose to use [Eleventy](https://www.11ty.dev/), a static site generator built for Node.js. While static site generators are a relatively recent phenomenon, the HTML which they produce is sound if authored correctly. The easy customizeablilty available in the Node.js environment means that new requirements are easily accounted for. Finally, many other static site generators use the same Markdown and YAML front matter format. 

This is something which seems to be commonly neglected in academia. Academics have traditionally been quite attached to printing--the quantity of material which we must read dictates that we prefer print since it causes less strain on the eyes. In addition, academics tend to have some technical knowledge, but only as a means to our studies. This means that we are often able to build websites, but rarely take the time to build them carefully since we are focusing on our studies. This series seeks to make lower this barrier. 

## Data Structure
Academic journals tend to be structured in issues and volumes. This is somewhat different from how websites tend to be structured: websites tend to be organized either by individual articles, by topic, by author, or by date. In addition, issues have a standard format. 

I chose to organize the files in a directory structure in order to keep them organized for future authors. It resembles this:
```
articles
  └───volume number
      └───issue number
```

The URLs follow a similar format: `/volume/issue/article-title/`. I chose to put the full permalink into each article's [front matter](https://www.11ty.dev/docs/data-frontmatter/). While it would be easy to generate these permalinks automatically, I wanted each article's permalink to be hard-coded to it. Thus, even if the content is re-organized, the content would be permanently accessible at the same URL as long as the new system uses the front matter data.[^1] 

The front-matter follows this format: 
```yaml
---
author: Author Name
authordescription: This is a brief biography of the author.
title: Title
tags: ["category"]
permalink: "/volume/issue/article-title/"
---
```

Each issue is represented by a JSON file. Eleventy has a [directory data file structure](https://www.11ty.dev/docs/data-template-dir/), by which data values can be set for the entire directory. Each file must be named after its directory: `/volume/issue/issue.json`. I used this to set article and volume information for all the articles, but also to provide data to generate a page for each issue. An issue data file is formatted like this:
```json
{
  "volume": 1,
  "issue": 1,
  "date": "date",
  "issueTitle": "Title of the Issue",
  "articleCategories": [
    "category 1",
    "category 2"
  ],
  "editors": [
    {
      "role": "Editor-in-Chief",
      "name": "Editor Name"
    },
    {
      "role": "Content Editor",
      "name": "Editor Name"
    }
  ]
}
```

The academic journal format also requires a page for each issue. I chose not to make a page for each volume because I wished to keep the site somewhat simpler. If a journal published many issues for each volume or each volume was thematically related, then having a page for each volume would be useful. The best way to generate a page for each issue is with the [pagination](https://www.11ty.dev/docs/pagination/) feature which Eleventy provides. However, each of these data files is its own separate file. They must be collected and added together into an array. Eleventy also allows for using [JavaScript as data files](https://www.11ty.dev/docs/data-js/), which allows making a script which collects all of these different data files into a single array. [The code is located here](https://github.com/westmarchjournal/website/blob/master/_data/fetchIssues.js). The pagination template looks like this. I have decided to leave the template code similar to the code used in Eleventy examples when possible (hence, `post.variables`). 
```liquid
---
pagination:
  data: fetchIssues
  alias: issue
  size: 1
tags: issue
eleventyComputed:
  title: "Journal vol. {{ issue.volume }}, no. {{ issue.issue }}: {{ issue.issueTitle }}"
permalink: "/{{ issue.volume }}/{{ issue.issue }}/"
---

<h1>Journal vol. {{ issue.volume }}, no. {{ issue.issue }}: {{ issue.issueTitle }}</h1>
{% for category in issue.articleCategories %}
  <h2>{{ category | capitalize }}</h2>
  {% set postslist = collections[ category ] %}
	 <ul reversed class="postlist">
	  {% for post in postslist | reverse %}
		  {% if post.data.volume == issue.volume and post.data.issue == issue.issue %}
		    <li class="postlist__item">
		      <a href="{{ post.url | url }}" class="postlist__item__link">
			      {% if post.data.title %}{{ post.data.title | safe }}, 
			      {{ post.data.author }}
			  </a>
		    </li>
		  {% endif %}
	  {% endfor %}
	</ul>
{% endfor %}

<h2>Editorial Staff</h2>
<dl class="editor-list">
  {% for editor in issue.editors %}
    <dt>{{ editor.role }}</dt>
    <dd>{{ editor.name }}</dd>
  {% endfor %}
</dl>
```

[^1]:  See Tim Berners-Lee,  *[Cool URIs don't change](https://www.w3.org/Provider/Style/URI.html)*, 1998. 

# Citation Generator
The particular needs of each academic journal will dictate which citation formats are readily available. The citation can be readily integrated into the article template. Enclosing the citation in a `<details` element avoids cluttering the page. This is an example of a Turabian note. 
```nunjucks
<details class="citation">
    <summary>Cite this page</summary>
    <p>
        {{ author }},
        &ldquo;{{ title | safe }},&rdquo;
        <i>{{ metadata.title | title }}</i>
        {{ volume }}, no. {{ issue }} ({{ page.date | readableDate }}),
        {{ metadata.citationurl}}{{ page.url | url }}.
    </p>
</details>
```

It is also helpful to provide citations in [RIS format](https://en.wikipedia.org/wiki/RIS_(file_format), which is easily read by most popular reference management software. These files are generated for each article using [pagination](https://www.11ty.dev/docs/pagination/), with the data source being `collections.article`. The `absoluteUrl()` filter is from the [Eleventy RSS plugin](https://www.11ty.dev/docs/plugins/rss/), which provides filters which are useful both in RSS and elsewhere. 
```nunjucks
---
pagination:
  data: collections.article
  size: 1
  alias: citation
  addAllPagesToCollections: true
permalink: /{{ citation.url }}/citation.ris
---
TY  - EJOUR
A1  - {{ citation.data.citationauthor }}
DA  - {{ citation.date | year }}
PY  - {{ citation.date | year }}
J1  - {{ metadata.title | title }}
J2  - {{ metadata.short_title }}
JA  - {{ metadata.short_title }}
JF  - {{ metadata.title | title }}
L2  - {{ citation.url | url | absoluteUrl(metadata.url) }}
LK  - {{ citation.url | url | absoluteUrl(metadata.url) }}
T1  - {{ citation.data.title }}
T2  - {{ metadata.title | title }}
TI  - {{ citation.data.title }}
VL  - {{ citation.data.volume }}
IS  - {{ citation.data.issue }}
ER  - 
```
In the page template, a link to the citation RIS file can be generated with `{{ page.url }}citation.ris`.  Note that `A1` should be in the format "Surname, Given Name". I discuss [below](#Author-Names) how to do this. 
# Inclusion for Google Scholar
[The Google Scholar authors discuss more thoroughly what sorts of metadata are necessary for indexing.](https://scholar.google.com/intl/en/scholar/inclusion.html#indexing) I simply used their example. In Nunjucks, it looks like this: 
```nunjucks
<meta name="citation_title" content="{{ title }}">
<meta name="citation_author" content="{{ citationAuthor }}">
<meta name="citation_publication_date" content="{{ page.date | year }}">
<meta name="citation_journal_title" content="{{ metadata.title | title}}">
<meta name="citation_volume" content="{{ volume }}">
<meta name="citation_issue" content="{{ issue }}">
```
## Metadata types (for future research)
- Highwire Press tags (used in the Google Scholar example).
- Eprints tags
- BE press tags
- PRISM tags
- Dublin Core tags (Google discourages them).
# Miscellenea 
## Author Names
Depending on the location, author names may need to be shown as "Firstname Surname" or "Surname, Firstname." For this, I stored the authors as both `name` and `surname`, and then used [Eleventy computed data](https://www.11ty.dev/docs/data-computed/) in the default template to form two variables, `author` (Given Name, Surname) and `citationauthor` (Surname, Given Name). I used conditionals because pseudonymous and collective authors will still be represented as `author`, because they do not have proper given names and surnames ([example](https://westmarchjournal.org/3/2/the-white-wood-2/)). 
```nunjucks
---
eleventyComputed:
    author: "{% if name and surname %}{{ name }} {{ surname }}{% else %}{{ author }}{% endif %}"
    citationAuthor: "{% if name and surname %}{{ surname }}, {{ name }}{% else %}{{ author }}{% endif %}"
---
```

### Including HTML elements in titles
In academic writing, it is often necessary to include italics in the title ([example](https://westmarchjournal.org/3/2/georgics-2-475-486/)). The `title` tag, however, does not allow child elements. It is thus necessary to use the Eleventy `safe` filter for the headings to allow HTML elements and to make a custom filter to remove HTML tags. This filter is also useful in feeds. The filter function should be [included in the Eleventy configuration file](https://www.11ty.dev/docs/filters/). 
```js
const striptags = require("striptags");
module.exports = function(eleventyConfig) {
  eleventyConfig.addFilter("striptags", function(value) {
      let product = striptags(value);
      return product;
  });
};
```

### Footnotes
Footnotes are necessary for academic writing. [`markdown-it-footnotes`](https://github.com/markdown-it/markdown-it-footnote) is a footnote plugin for `markdown-it`, and is compatible with Pandoc. The Eleventy documentation discusses [how to use markdown plugins](https://www.11ty.dev/docs/languages/markdown/#add-your-own-plugins). 
