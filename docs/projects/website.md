# My Website
Across my lifetime, I have tried numerous mechanisms for my website with a basic blog system, including [WordPress](https://wordpress.org), [Pelican](https://blog.getpelican.com), [Jekyll](https://jekyllrb.com/), and even a home-grown solution on [Google App Engine](https://cloud.google.com/appengine/). I have now settled on [MkDocs](https://www.mkdocs.org/), which is definitely not intended for blogging, but I find it well-suited for the task.
## Getting Started
I installed mkdocs with `#!bash pip install mkdocs` and created a new project using `#!bash mkdocs new .`. I also initialised a [Git repository on GitHub](https://github.com/retnikt/retnikt.uk) to version all the configuration and content.
## Customisation
Now it's time for some configuration. 
### Theme
The first thing to do is pick a theme. I chose the popular and brilliant [mkdocs-material](https://squidfunk.github.io/mkdocs-material/).
![mkdocs-material](https://i.imgur.com/BjcuAVf.png)
To install, you can follow [this guide](https://squidfunk.github.io/mkdocs-material/) for detailed instructions, but essentially: you use pip. (`#!bash pip install mkdocs-material`)

To enable this theme, we need to add it to the mkdocs.yml configuration file:
???+ abstract "Code"
    ```yaml
    theme:
      name: 'material'
      custom_dir: 'theme'
    ```
The `custom_dir` line is so that we can override parts of the HTML.

We can make some tweaks to the rest of the appearance too, including colours, fonts, and [many other options](https://squidfunk.github.io/mkdocs-material/getting-started/#configuration). Here's my final `theme` section:

???+ abstract "Code"
    ```yaml
    theme:
      name: 'material'
      custom_dir: 'theme'
      language: en
      palette:
        primary: 'blue'
        accent: 'blue'
      font:
        text: 'Noto Sans'
        code: 'Fira Code'  # we'll need some extra setup to make this work
      logo:
        icon: 'developer_board'
      feature: false
      favicon: 'assets/images/favicon2.png'
    ```


### Extra Options
First let's set some basic options:

???+ abstract "Code"
    ```yaml
    site_name: retnikt
    site_url: 'https://retnikt.uk/'
    ```

I'm going to add a copyright to the footer and some social media links, and enable a few (lots of) Markdown extensions:

???+ abstract "Code"
    ```yaml
    copyright: '© retnikt 2019'
    extra:
      social:
        - type: 'github'
          link: 'https://github.com/retnikt'
        - type: 'twitter'
          link: 'https://twitter.com/retnikt'
    markdown_extensions:
      - admonition
      - codehilite
      - footnotes
      - meta
      - pymdownx.arithmatex
      - pymdownx.betterem:
          smart_enable: all
      - pymdownx.caret
      - markdown.extensions.attr_list
      - markdown.extensions.def_list
      - markdown.extensions.tables
      - markdown.extensions.abbr
      - pymdownx.extrarawhtml
      - pymdownx.critic
      - pymdownx.details
      - pymdownx.emoji:
          emoji_generator: !!python/name:pymdownx.emoji.to_svg
      - pymdownx.inlinehilite
      - pymdownx.magiclink
      - pymdownx.mark
      - pymdownx.smartsymbols
      - pymdownx.superfences
      - pymdownx.progressbar
      - pymdownx.keys
      - pymdownx.tasklist:
          custom_checkbox: true
      - pymdownx.tilde
      - toc:
          permalink: true
    ```
### Block Overrides
I want to customise some of the theme components, particularly the footer. I'm going to add a GitHub Pages reference (see [Deploying](#deploying) below), and hide the Next/Previous arrows for the homepage. Here's what I changed in the `partials/footer.html` Jinja2 template.

???+ abstract "Code"
    ```html
      ...
      <!-- if this is the homepage, then don't display the arrows -->
      {% if (not page.is_homepage) and (page.previous_page or page.next_page) %}
        <div class="md-footer-nav">
          <nav class="md-footer-nav__inner md-grid">
            <!-- if previous page is the homepage, then don't display the previous arrow -->
            {% if page.previous_page and not page.previous_page.is_homepage %}
            ...
            <a href="https://www.mkdocs.org">MkDocs</a>
            and
            <a href="https://squidfunk.github.io/mkdocs-material/">
              Material for MkDocs</a>
            hosted on
            <a href="https://pages.github.com">
              GitHub Pages</a>
          </div>
          ...
    ```
I also modified `partials/nav.html` as well, changing `{{ config.site_name }}` (which would contain the site name) to just "Navigation" and change "Table of contents" to just "Contents" with a modified language file (`partials/language/en.html`).

Also, to get the Fira Code font to work we need to modify the `fonts` block in the HTML `#!html <head>`. In `main.html`, add:

???+ abstract "Code"
    ```html
    <!-- override the fonts block -->
    {% block fonts %}
      <style>
      /* use the Fira Code font from CDN */
      @font-face{
        font-family: 'Fira Code';
        src: local('Fira Code Medium'), local('Fira Code'),
        url('https://cdn.jsdelivr.net/npm/firacode@1.206.0/distr/eot/FiraCode-Medium.eot');
        src: url('https://cdn.jsdelivr.net/npm/firacode@1.206.0/distr/eot/FiraCode-Medium.eot') format('embedded-opentype'),
            url('https://cdn.jsdelivr.net/npm/firacode@1.206.0/distr/woff2/FiraCode-Medium.woff2') format('woff2'),
            url('https://cdn.jsdelivr.net/npm/firacode@1.206.0/distr/woff/FiraCode-Medium.woff') format('woff'),
            url('https://cdn.jsdelivr.net/npm/firacode@1.206.0/distr/ttf/FiraCode-Medium.ttf') format('truetype');
        font-weight: 500;
        font-style: normal;
      }
      </style>
      {% if font != false %}
        <link href="https://fonts.gstatic.com" rel="preconnect" crossorigin>
        <!-- note the change to this link tag to stop trying to load Fira Code from Google Fonts (it's not there) -->
        <link rel="stylesheet" href="https://fonts.googleapis.com/css?family={{
              font.text | replace(' ', '+')  + ':300,400,400i,700|'
            }}">
        <!-- note the change to this style tag to load Fira Mono as a backup -->
        <style>body,input{font-family:"{{ font.text }}","Helvetica Neue",Helvetica,Arial,sans-serif}code,kbd,pre{font-family:"{{ font.code }}","Fira Mono","Courier New",Courier,monospace}</style>
      {% endif %}
    {% endblock %}
    ```
All these modified files are available on my [GitHub](https://github.com/retnikt/retnikt.uk) repo for this project.
## Deploying
Following the guide from [MkDocs](https://www.mkdocs.org/user-guide/deploying-your-docs/), I deployed to GitHub Pages with a custom domain ([retnikt.uk](https://retnikt.uk/)) which you have to put in a CNAME file in your `docs/` folder. Every time I  make a change I simply run `mkdocs gh-deploy`, which does all the work for you.
### Continuous Integration
In the future I hope to add some continuous integration to the building and deployment of the website, a topic I am totally new to. Look out for updates in the future!
