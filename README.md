# General Principle
Multi-language documentation is actually created using three separate MkDocs builds, one for each language. This has some consequences for the setup:

- Three configuration files
- Three folders for markdown files in the `docs/` folder
- Three folders for the HTML in the output folder `generated/`
- Optional: in the `includes/` folder, three files for the URLs used in the markdown.

Shared files - such as the logo, favicon and stylesheets - are stored in the `overrides/assets/` folder.

One small disadvantage is that you cannot use the `mkdocs serve` command to serve your site locally with all three languages. MkDocs can only serve one language at the time.

# File Structure
The project directory has the following structure:
```
.
├─ config/
│    ├─ en/
│    │  └─ mkdocs.yml
│    └─ es/
│        └─ mkdocs.yml
│
├─ generated/
│    ├─ en/
│    └─ es/
│
├─ includes/
│    ├─ developers/
│    │   └─ api.md
│    │
│    ├─ urls-en.md
│    └─ urls-es.md
│
├─ docs/
│    ├─ en/
│    │   ├─ index.md 
│    │   └─ developers.md
│    └─ es/
│        ├─ index.md 
│        └─ developers.md
│
└─ overrides/
     ├─ assets/
     │   ├─ images/
     │   │    ├─ favicon.ico
     │   │    └─ logo.svg
     │   │
     │   └─ stylesheets/
     │        └─ yesplan.css
     │
     └─ partials/
              └─ ...
```
# Configuration Files
In the three configuration files, you can set options for the separate languages. Below, you will find slimmed-down examples for English and Dutch. We use many more extensions and plugins, but they are irrelevant for the recipe.

Some things to bear in mind:

- It is hard to maintain the same CSS or images in three locations (`docs/es/` and `docs/en/`). Therefore, the logo, favicon and CSS are shared by all the languages and have been moved to the `overrides/assets/` folder. This configuration deviates from the main Material for MkDocs documentation.
- The `language` key is used for:
 
  - The value of the `lang` attribute in the `html` tag
  - The buttons "Previous", "Next" etc. (see [Changing the language](https://squidfunk.github.io/mkdocs-material/setup/changing-the-language/))
  - The language selection in the header (see below)
  - Conditions that check the site's current language, for instance to show an announcement in the correct language (see below).
- For the navigation, you should simply add the path starting from the folder set in the `docs_dir` key. So in this case, use `index.md` and not `docs/en/index.md`.

## English
```yaml
site_name: 'Manual'
docs_dir: '../../docs/en'                           # Where to find the English markdown files
site_dir: '../../generated/en'                      # Where to put the English HTML files

theme:
    name: material
    custom_dir: '../../overrides/'                  # This is where the customization of the theme lives
    logo: assets/images/logo.svg                    # The logo is shared by all languages
    favicon: assets/images/favicon.ico              # The favicon is shared by all languages
    language: en                                    # The build's language

extra_css:
  - assets/stylesheets/yesplan.css                  # CSS is shared by all languages

extra:                                              # Language Selection
  alternate:  

      # Switch to English
    - name: English
      link: /en/
      lang: en

    # Switch to Spanish
    - name: Español
      link: /es/
      lang: es

plugins:
  - search:                                         
      lang: en                                      # Set language for search

nav:
- Welcome: index.md
```

## Spanish
```yaml
site_name: 'Manual'
docs_dir: '../../docs/en'                           # Where to find the English markdown files
site_dir: '../../generated/en'                      # Where to put the English HTML files

theme:
    name: material
    custom_dir: '../../overrides/'                  # This is where the customization of the theme lives
    logo: assets/images/logo.svg                    # The logo is shared by all languages
    favicon: assets/images/favicon.ico              # The favicon is shared by all languages
    language: es                                    # The build's language

extra_css:
  - assets/stylesheets/yesplan.css                  # CSS is shared by all languages

extra:                                              # Language Selection
  alternate:  

    # Switch to English
    - name: English
      link: /en/
      lang: en

    # Switch to Spanish
    - name: Español
      link: /es/
      lang: es

plugins:
  - search:                                         
      lang: es                                      # Set language for search

nav:
- Welcome: index.md
```

# Language Selection
Material for MkDocs offers a nice solution out of the box from version 7.0.0 onwards: see [Site Language Selector](https://squidfunk.github.io/mkdocs-material/setup/changing-the-language/#site-language-selector) for more information. The examples above use the native Material for Mkdocs selector.

You can also extend the theme and use a condition like the one for announcements to build your own selector. In that case, be very careful when upgrading mkdocs-material, since the code you are overriding may change over time.

# Language Detection

What if you want to use [announcements](https://squidfunk.github.io/mkdocs-material/setup/setting-up-the-header/#announcement-bar)? They are set globally by extending the theme, but you want your announcement to appear in the correct language, right? In that case you can detect the language using a condition:
```
{% block announce %}
{% if config.theme.language == 'nl' %}
Welkom!
{% elif config.theme.language == 'en' %}
Welcome!
{% else %}
Bienvenue !
{% endif %}
{% endblock %}
```

# Putting it all together
Creating the HTML for these files is now simply a matter of executing three commands. If you are in the root of your project:
```
mkdocs build -f config/en/mkdocs.yml
mkdocs build -f config/fr/mkdocs.yml
mkdocs build -f config/nl/mkdocs.yml
```
You can then push the generated files to a web server. In the past, we used [ant](https://ant.apache.org/manual/) to execute the tasks, but we now use [Docker](https://www.docker.com/).

# Shared Content
Sometimes, content may be shared across all languages. For instance, our documentation for developers is always in English and should appear in the sections for all three languages: English, French and Dutch. However, I don't want to maintain the same content in three places, since this will certainly lead to errors.

## Simple

If this documentation does not contain URLs to other language-specific files (e.g. `/en/what-is-an-event.md`), you can simply use Snippets.

For instance, in the `docs/en/` and `docs/es/` folder, you can add a file `developers.md` with the following contents:
```
--8<-- "../../includes/developers/api.md"
```

This will include the one file in all three languages. You only have to maintain the file includes/developers/api.md and it will appear correctly in all languages. Note that the path you use for the snippet is relative to the mkdocs config file.

## Not That Simple
In our case, the situation was a bit more complex. The shared content for developers contained many links to documentation in the different languages, e.g. to refer to concepts, actions, interfaces etc. If we included one file with hard-coded URLs, these links would always refer to one single language.

This was solved by using reference links in a separate file and including it with snippets (see "Managing URLs" below).

For English, the file `developers.md` would look like this:

```
--8<-- "../../includes/developers/api.md"

--8<-- "../../includes/urls-en.md
```
But for Spanish, you include a different URL file:
```
--8<-- "../../includes/developers/api.md"

--8<-- "../../includes/urls-es.md
```
That way, you only have to maintain one file (api.md), but you can still use different URLs for every language.

# Managing URLs (Optional)

More languages means more URLS: currently, our documentation has about 600 URLs across all languages. This is quite complex:

If you change a (sub)heading, you also change the anchor to that heading, resulting in a broken link. But does that link already exist in the documentation?
URLs are confusing for translators. Should they also change the URL when translating? What should the new URL be then? And if you receive the translated files, you have to make sure that every URL in every translated file or section is correct.
Needless to say, URLs are a nightmare to maintain, and there is no ideal solution.

Setup for URLS
I decided to use reference links in the markdown, e.g. `[Go home][1]`. At the bottom of every markdown file, I include a file containing all the keys, e.g. `--8<-- "../../includes/urls-en.md"`. See [Snippets](https://squidfunk.github.io/mkdocs-material/reference/abbreviations/#snippets) for more information on this feature.

For English, the URLs file looks like this:
```
[1]: /en/
[2]: /en/contact-yesplan/
```
For Spanish, the URLs file looks like this:
```
[1]: /es/
[2]: /es/contact-yesplan/
```

- Advantages

    - Every language has its own file.
    - Only one place to maintain all URLs for each language.
    - In the markdown, the URLs are replaced by keys that are the same across all languages.
    - Translators don't have to translate URLs, leading to fewer errors.
    - This is part of the solution for shared content with different links (see above).
    
- Disadvantages

    - When maintaining the documentation, you don't see what hyperlinks refer to. You always have to go to the URLs file.
    - You have to use absolute links, bypassing the tests MkDocs executes when building the documentation.

# Testing the URL Files
I wrote a Perl script to run some tests and create a markdown file containing all the links. I'm not a programmer, so I think that anyone with basic programming skills can do this. I'm not sharing the script here because it is very specific to our setup.

The file checks the following things for every language:

- Do the markdown files in the `docs/` folder contain empty keys, e.g. `[Some link][]`?
- Do the markdown files in the `docs/` folder contain "orphan" keys that are not in the URLs files?
- Do the URL files contain any "childless" keys that are not in the markdown files?
- Does a URL file contain duplicate keys?
- Does a URL file contain duplicate URLs?

There is also one test across languages:

- Is there a difference in key frequency between languages? For instance, if the key `[1]` appears three times for English, but only once for Dutch, there may be a problem with the translation.

# Testing the URLs
Once all this is checked, the script creates a markdown file for each language in the `docs/` folder.

For English, the file `docs/en/test-links.md` may look like this:
```
# Check Links EN

- [1][1]
- [2][2]

--8<-- "../../includes/urls-en.md"
```
When the documentation has been built in Docker or on our development server, I browse to that page and run a link checker:

- Since the file contains all the site's links for that language, I check all links with one push of the button.
- I use the Chrome extension [Check My Links](https://chrome.google.com/webstore/detail/check-my-links/ojkcdipcgfaekbeaelaapakgnjflfglf). It detects broken links (error 404), but also broken links to anchors inside pages (code 200 but still a warning).
- Since the text for the hyperlink contains the key, I immediately know which one to check.

These test files are excluded from the build for the production server.

[Original source](https://github.com/squidfunk/mkdocs-material/discussions/2346#discussion-3240838)