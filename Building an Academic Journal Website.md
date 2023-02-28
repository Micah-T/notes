
# Introduction and Requirements
I recently built the website for the [Westmarch Literary Journal](https://westmarchjournal.org/) at Patrick Henry College. This is an academic journal website, so there are some requirements which are not commonly found in other websites. 

## Maintainability and Readibility
Academic journal websites must be readible and maintainable for a long period of time. Academic study seeks to increase the sum of knowledge that is available while also building certainty in this knowledge. To increase the sum of available knowledge requires readibility--our work is not particularly useful if it is inaccessible. To make the knowledge more certain requires maintaining this access--if cited works can no longer be found, this makes verifying research impossible. 

Readibility requires making websites easily accessible. This is a complex and frequently dicussed topic, though it can be simply summed: one ought to avoid doing anything which makes a website hard to read; that is, websites should be no more complex for the user's computer than is necessary. Thus, a correctly-authored HTML with few scripts is how an academic website should be authored. 

Maintainability requires that the content be easily updated and maintained for a long time. The website author must consider what format the content is stored in--if the system which processes the content becomes unavailable, the content must be easily converted to another format. I chose to use Markdown to store the content for this reason. [Pandoc](https://pandoc.org/) allows for easy conversion to and from Markdown, and Markdown itself is easily readible in a text editor. In addition, many other web publishing tools natively support Markdown. I also chose to use [Eleventy](https://www.11ty.dev/), a static site generator built for Node.js. While static site generators are a relatively recent phenomenon, the HTML which they produce is sound if authored correctly. The easy customizeablilty available in the Node.js environment means that new requirements are easily accounted for. Finally, many other static site generators use the same Markdown and YAML front matter format. 

This is something which seems to be commonly neglected in academia. Academics have traditionally been quite attached to printing--the quantity of material which we must read dictates that we prefer print since it causes less strain on the eyes. In addition, academics tend to have some technical knowledge, but only as a means to our studies. This means that we are often able to build websites, but rarely take the time to build them carefully since we are focusing on our studies. This series seeks to make lower this barrier. 

## Structure
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
  "volume": volume number (integer),
  "issue": issue number (integer),
  "date": "data",
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

The academic journal format also requires a page for each issue. I chose not to make a page for each volume because I wished to keep the site somewhat simpler. If a journal published many issues for each volume or each volume was thematically related, then having a page for each volume would be useful. The best way to generate a page for each issue is with the [pagination](https://www.11ty.dev/docs/pagination/) feature which Eleventy provides. However, each of these data files is its own separate file. They must be collected and added together into an array. Eleventy also allows for using [JavaScript as data files](https://www.11ty.dev/docs/data-js/), which allows making a script which collects all of these different data files into a single array. [The code is located here](https://github.com/westmarchjournal/website/blob/master/_data/fetchIssues.js). The pagination template looks like this. I have decided to leave code similar to normal website code when possible (hence, `post.variables`). 
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

## Really Specific Things
HTML tags in titles
footnotes
