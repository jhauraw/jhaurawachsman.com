+++
date = "2020-02-23T04:30:00Z"
lastmod = "2020-02-24T21:00:00Z"
author = "jhaurawachsman"
title = "Hugo Atom Syndication XML Template (Better RSS)"
subtitle = "Supercharge your Hugo website's syndication feed with an Atom Feed XML template. Here's how to make the switch from RSS 2.0 to Atom in Hugo."
feature = "hugo-atom-syndication-xml-template.webp"
+++

The Atom Syndication feed format is an alternative to the more common and well-known RSS 2.0 format. The Atom protocol provides a number of advantages over is older rival, making it a worthwhile addition to your Hugo website. To make the switch in Hugo we need to create an Atom specific XML feed template and link it in our theme's HTML Head section.  

# Preparation
## New Hugo Site

By way of example, let's create a fresh Hugo website installation:

> When typing/copying the terminal commands below, don't include the prompt portion: `~ %`

```shell
# Assuming a working directory named "sites"
~ % cd sites/
sites % hugo new site hugo-atom
sites % cd hugo-atom/

# Add a theme
hugo-atom % git init
hugo-atom % git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke

# Add theme to config
hugo-atom % echo 'theme = "ananke"' >> config.toml

# Add two "Posts"
hugo-atom % hugo new posts/first-post.md
hugo-atom % hugo new posts/second-post.md
```

Now that we have a fresh working environment and a couple of posts, we can begin to implement our Atom Feed XML template for our Hugo website.

## Assumptions / Opinions

For this example, let's consider the common "Blog" setup, with a single section called "Posts" (or "Blog", or "Articles").

Furthermore, we will go with these opinions:
- We only want one main feed and it's located in site public root
- Only "Posts" should show up in the feed, not static pages, such as About, Contact, or Privacy
- Full article content should be in the feed, not just a summary
- Our feed should be auto-discoverable and linked in the HTML Head section (`<head>`) of our theme

> See below, for how the feed template can be modified to support multiple Sections, such as "Posts" and "Projects".

# Setup Steps
## Add Hugo Config Settings

Open the Hugo _Config_ file (`hugo-atom/config.toml`) in your favorite editor and add these settings, being careful not to delete its current contents:

```toml
# RSS limit, remove for unlimited posts
rssLimit = 100

[outputs]
# Output HTML, and ATOM on Home
home = ["HTML", "ATOM"]
# Output only HTML everywhere else
section = ["HTML"]
taxonomy = ["HTML"]
taxonomyTerm = ["HTML"]

# Define a new ATOM output format
[outputFormats]
[outputFormats.ATOM]
name = "ATOM"
baseName = "feed"
mediaType = "application/atom+xml"

# Define a new ATOM media type
[mediaTypes]
[mediaTypes."application/atom+xml"]
suffixes = ["atom"]

# Define the main Sections
[params]
mainSections = ["posts"]

# Example Brand settings
[params.brand]
tagline = "Publisher's Tagline"
icon = "icon.png"
logo = "logo.png"
```

## Authors Data File

Create an _Authors_ file (`authors.toml`) in the _Data_ folder (`hugo-atom/data`):

> TOML is used here, but you can use YAML, JSON, or any of the supported Data file formats.

```shell
hugo-atom % touch data/authors.toml
```

Open the  _Authors_ file (`authors.toml`) in your editor and copy/paste this data:

```toml
[default]
name = "Site Publisher"
uri = "https://www.publisher.com/"
email = "contact@publisher.com"
twitter = "publisher"
image = "default.png"

[joesmith]
name = "Joe Smith"
uri = "https://www.joesmith.com/"
email = "joe@joesmith.com"
twitter = "joesmith"
image = "joesmith.png"
```

Open the _First Post_ file (`hugo-atom/content/posts/first-post.md`) in your editor, and add an _Author Id_ in the front matter to attribute the post to _Joe Smith_, and a wee bit of content:

```yaml
---
author: joesmith
---

First Post.
```

## Atom Feed Layout Template

Create an _Atom Feed_ template file (`index.atom`) for your Hugo website in the _Layouts_ folder (`hugo-atom/layouts`):

```shell
hugo-atom % touch layouts/index.atom
```

Open the new _Atom Feed_ file (`index.atom`) in your editor, and copy/paste this code:

```xml
{{- $pages := where .Site.RegularPages "Type" "in" .Site.Params.mainSections -}}
{{- $limit := .Site.Config.Services.RSS.Limit -}}
{{- if ge $limit 1 -}}
{{- $pages = $pages | first $limit -}}
{{- end -}}
{{ printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\"?>" | safeHTML }}
<feed xmlns="http://www.w3.org/2005/Atom" xml:lang="{{ .Site.LanguageCode }}">
  <title>{{ .Site.Title }}</title>
  {{- with .Site.Params.brand.tagline }}
  <subtitle>{{ . }}</subtitle>
  {{- end }}
  <id>{{ "/" | absLangURL }}</id>
  <author>
    <name>{{ .Site.Title }}</name>
    <uri>{{ "/" | absLangURL }}</uri>
  </author>
  <generator>Hugo gohugo.io</generator>
  {{- with .Site.Copyright }}
  <rights>{{ . }}</rights>
  {{- end }}
  {{- with .Site.Params.brand.icon }}
  <icon>{{ . | absURL }}</icon>
  {{- end }}
  {{- with .Site.Params.brand.logo }}
  <logo>{{ . | absURL }}</logo>
  {{- end }}
  <updated>{{ dateFormat "2006-01-02T15:04:05Z" now.UTC | safeHTML }}</updated>
  {{- with .OutputFormats.Get "ATOM" }}
  {{ printf `<link rel="self" type="%s" href="%s" hreflang="%s"/>` .MediaType.Type .Permalink $.Site.LanguageCode | safeHTML }}
  {{- end }}
  {{- range .AlternativeOutputFormats }}
  {{ printf `<link rel="alternate" type="%s" href="%s" hreflang="%s"/>` .MediaType.Type .Permalink $.Site.LanguageCode | safeHTML }}
  {{- end }}
  {{- range $pages }}
  <entry>
    <title>{{ .Title }}</title>
    {{- $author := index .Site.Data.authors (.Params.author | default "default") }}
    <author>
      <name>{{ $author.name }}</name>
      <uri>{{ $author.uri }}</uri>
    </author>
    <id>{{ .Permalink }}</id>
    {{- if .IsTranslated -}}
    {{ range .Translations }}
    <link rel="alternate" href="{{ .Permalink }}" hreflang="{{ .Language.Lang }}"/>
    {{- end -}}
    {{ end }}
    <updated>{{ dateFormat "2006-01-02T15:04:05Z" .Lastmod.UTC | safeHTML }}</updated>
    <published>{{ dateFormat "2006-01-02T15:04:05Z" .Date.UTC | safeHTML }}</published>
    <content type="html">{{ trim .Content "\n" }}</content>
  </entry>
  {{- end }}
</feed>
```

# Testing it Out
## Build the Hugo Website

Run this simplified Hugo command in your terminal app to build the website static files into the _Public_ directory (`hugo-atom/public`):

```shell
hugo-atom % hugo --buildDrafts --cleanDestinationDir
```

## Examine the Atom Feed XML

If everything went according to plan, you should see the generated _Atom Feed_ file (`feed.atom`) in the _Public_ directory (`hugo-atom/public/feed.atom`). Open it up in your editor and take a look at the output to verify that everything looks correct.

> Notice how the _Second Post_ entry has the _Default_ author's details, while the _First Post_ entry has _Joe Smith's_ details? Because we specified him by Id in the _First Post's_ front matter, he is shown as the _Author_.

## Validate the Atom Feed XML

When you feel the Atom Feed XML output is production ready, copy/paste the contents of the generated file and validate it via direct input using the [W3C Feed Validator Service](https://validator.w3.org/feed/#validate_by_input).

> When validating via direct input ignore any "self reference doesn't match document location" or "content should not be blank" errors. Later, validate your feed by URL once deployed live and these errors shouldn't be present.

# Atom Feed Auto-Discovery
## Add a Link Element in Your Theme

As a final step, we need to let the world know we have an Atom Feed available for our "Posts" Section. We do this with a _Link_ element (`<link>`) to our Atom Feed in the _Head_ section (`<head>`) of our theme's _BaseOf_ file (`baseof.html`).

In an earlier step, we added a new Atom output format (`outputFormats.ATOM`) in our config file. Some themes may not automatically add the Link element in the Head section, so we may need to set that up. This is true of the default Hugo theme (`ananke` v2.54), as it uses an RSS specific method of looking for alternative output formats. To fix this, we need to replace that code snippet with a more up-to-date method. To do this:
- Copy the Ananke theme's _BaseOf_ file (`hugo-atom/themes/ananke/layouts/_default/baseof.html`)
- Paste the file in the site root _Layouts_ folder (`hugo-atom/layouts/_default/`)

Next, open the pasted _BaseOf_ file in your editor and replace the code snippet at `line: 30`:

```xml
{{ if .OutputFormats.Get "RSS" }}
{{ with .OutputFormats.Get "RSS" }}
  <link href="{{ .RelPermalink }}" rel="alternate" type="application/rss+xml" title="{{ $.Site.Title }}" />
  <link href="{{ .RelPermalink }}" rel="feed" type="application/rss+xml" title="{{ $.Site.Title }}" />
  {{ end }}
{{ end }}
```

Replace the entire block above with this snippet:

```xml
{{- range .AlternativeOutputFormats }}
{{ printf `<link rel="%s" type="%s" href="%s" hreflang="%s"/>` .Rel .MediaType.Type .Permalink $.Site.LanguageCode | safeHTML }}
{{- end }}
```

Run the simplified Hugo build command again:

```shell
hugo-atom % hugo --buildDrafts --cleanDestinationDir
```

Open the website's generated Home page (`hugo-atom/public/index.html`) in your editor, and look down at `line: 25`, you should see the _Link_ element (`<link>`) with an HREF (`href`) pointing your Atom Feed XML file's live public URL (`http://example.org/feed.atom`):

```html
<link rel="alternate" type="application/atom+xml" href="http://example.org/feed.atom" hreflang="en-us"/>
```

## Hurray!

If you see the output above, congratulations, you have successfully implemented the Atom Feed Syndication protocol and Auto-discovery on your Hugo website!

# Key Elements of the Template Explained
## Include Only Pages from the Posts Section

The snippet below is responsible for pulling in `RegularPages` from the main Sections. This excludes static pages like About, Contact, or Privacy, and list pages like a "Posts" archive index:

```xml
{{- $pages := where .Site.RegularPages "Type" "in" .Site.Params.mainSections -}}
```

> Tip: You can modify this snippet to include entries from multiple Sections such as "Posts" and "Projects", by editing the `where` clause.

## Limiting the Number of Entries

If the config file setting `rssLimit` is set, we respect that with this snippet:

```markup
{{- $limit := .Site.Config.Services.RSS.Limit -}}
...
```

## Full Post Content, Not Just Summary

Users reading your content via feed aggregation services such as Feedly, will thank you for generating your feed with full post content:

```xml
<content type="html">{{ trim .Content "\n" }}</content>
```

# Example Project Source Code
## GitHub Repository

For reference, I've created a GitHub repository with the source code used in this article, at: <https://github.com/jhauraw/hugo-atom-xml-template>

# Parting Thoughts
## Getting the Right Fit

It's likely your site directory structure will differ. Hugo certainly allows for innumerable setups. However, hopefully you can take the core tweaks outlined here and spin up a supercharged Hugo Atom Syndication XML Feed that works for your needs.

The Atom feed format provides a number of advantages over RSS 2.0 and making the switch is well worth the effort. Remember, make sure to validate your feed before going live!
