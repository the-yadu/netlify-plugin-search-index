# Netlify Search Index Plugin

Generate a Search Index you can query via JavaScript or a Netlify Function!

You may not need this - There are other ways to add search to your site, like using Algolia or [Vanilla JS with a custom search Index](https://www.hawksworx.com/blog/adding-search-to-a-jamstack-site/).

However, you may wish to have a way to generate this index based ONLY on crawling your generated static site, or you may wish to do **index searches in a serverless function** instead of making your user download the entire index and run clientside.

## Usage

Specify the plugin in your `netlify.yml`. No config is required but we show the default options here.

**Generating both**:

```yml
build:
  functions: functions # must specify a functions folder for this to work
  publish: build # your normal netlify publish folder
  command: echo "your build command goes here" # your normal netlify build command

plugins:
  - package: netlify-plugin-search-index
    # all config is optional, we just show you the defaults below
    # config: 
      # generatedFunctionName: search # change the name of generated folder in case of conflicts, use `null` to turn off
      # publishDirJSONFileName: searchIndex # also use null to turn off

      # # optional configs from html-to-text - for explanation see https://www.npmjs.com/package/html-to-text#user-content-options
      # tables: []
      # wordwrap: null
      # linkHrefBaseUrl: http://asdf.com 
      # hideLinkHrefIfSameAsText: false 
      # noLinkBrackets: false
      # ignoreHref: false
      # ignoreImage: false
      # preserveNewlines: false
      # decodeOptions: ??
      # uppercaseHeadings: false
      # singleNewLineParagraphs: false
      # baseElement: body # useful to try article or main
      # returnDomByDefault: false
      # longWordSplit: null
      # format: text, image, lineBreak, paragraph, anchor, heading, table, orderedList, unorderedList, listItem, horizontalLine
      # unorderedListItemPrefix: ' * '

      # plugin debugging only
      debugMode: false # (for development) turn true for extra diagnostic logging
```

**Generating serverless function only**:

To use this plugin only for the generated serveless function, supply `null` to the `publishDirJSONFileName`:

```yml
plugins:
  - package: netlify-plugin-search-index
    config: 
      generatedFunctionName: mySearchFunction
      publishDirJSONFileName: null
```

This would generate a Netlify function at `https://yoursite.netlify.com/.netlify/functions/mySearchFunction` which you can query with `https://yoursite.netlify.com/.netlify/functions/mySearchFunction?search=foo`.

**Generating clientside JSON only**:

To use this plugin only for the clientside JSON file, supply `null` to the `generatedFunctionName`:

```yml
plugins:
  - package: netlify-plugin-search-index
    config: 
      generatedFunctionName: null
      publishDirJSONFileName: mySearchIndex # you can use / to nest in a directory
```

This would generate a clientside JSON at `https://yoursite.netlify.com/mySearchIndex.json`.

Supplying `null` to both would be meaningless.

## What It Does

After your project is built:

- this plugin goes through your HTML files
- converts them to searchable text with [`html-to-text`](http://npm.im/html-to-text)
- stores them as a JSON blob in `/searchIndex/searchIndex.json` (*You can customize the folder name in case of conflict*)
- generates a [Netlify Function](https://docs.netlify.com/functions/overview/?utm_source=twitter&utm_medium=laddersblog-swyx&utm_campaign=devex) that fuzzy searches against a query string with [fuse.js](https://fusejs.io/)

You can use this plugin in two ways:

- **Client-side**: You can simple require the JSON blob in your clientside JavaScript if it isn't too big:
    ```js
    // app.js
    import searchIndex from './searchIndex/searchIndex.json'
    ```
- **Serverless-side**: You can use the generated function that reads the JSON and returns fuzzy search results to be lighter on your frontend. The generated function is available at `.netlify/functions/searchIndex` and you can use it with a search term like `.netlify/functions/searchIndex?s=foo` or `.netlify/functions/searchIndex?search=foo`:
    ```js
    // app.js
    document.getElementById('myForm').addEventListener('submit', async event => {
      const result = await fetch(`/.netlify/functions/searchIndex?search=${event.target.searchText.value}`).then(x => x.json())
      document.getElementById('result').innerText = JSON.stringify(result, null, 2)
    })
    ```

Under the hood, the search function uses [fuse.js](https://fusejs.io/) and in future we may expose more configurations for this.

## Configuration

Plugin options:

- `searchIndexFolder` (default: `searchIndex`): where the plugin stores `searchIndex.json`.

## Notes for contributors

We had to use [patch-package](https://github.com/ds300/patch-package) to fix this bug: https://github.com/paulmillr/readdirp/issues/157 - `readdirp` is a dependency of ``copy-template-dir` which we use.

Hopefully it will be resolved or we can just fork `copy-template-dir` entirely locally.

## Future plans

WE ARE SEEKING MAINTAINERS.

- better html parsing - header tags and SEO metadata should be parsed too
- expose fuse.js and html parse search options for more configurability
