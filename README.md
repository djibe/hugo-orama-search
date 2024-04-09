# Hugo Orama search

An [Orama search](https://oramasearch.com) implementation for [Hugo](https://gohugo.io).

## 1. Create the database layout

We use Hugo's fast file generation to generate our index in JSON format.

In the `layouts` folder, create an `index.json.json` file.

```html
[ {{- $i := 0 -}}
  {{- range where .Site.RegularPages "Section" "ne" "" -}}
     {{- if not .Params.noSearch -}}
        {{- if gt $i 0 }},{{ end -}}
        {"uri":"{{ .RelPermalink }}", "title":{{ .Title | jsonify  }}, "content":{{ .Plain | htmlUnescape | jsonify }}, "tags": "{{ delimit .Params.tags ", " }}" }
        {{- $i = add $i 1 -}}
     {{- end -}}
  {{- end -}} ]
```

Now tell Hugo to generate the `index.json` from this template.

In you `config.toml`, add a `[outputs]` section (or edit it like this) to generate `index.html` (homepage) and our `index.json` (search index).

```toml
[outputs]
  home = ["HTML", "JSON"]
```

## 2. Import Orama in your footer

Import *Orama* before your `body` closing tag.

```html
<script type="module" async crossorigin>
  import {create, search, insertMultiple} from 'https://cdn.jsdelivr.net/npm/@orama/orama@2.0.15/dist/index.js';
  import {stemmer} from 'https://cdn.jsdelivr.net/npm/@orama/stemmers/dist/fr.js';
</script>
```

Now add this code within the module:

```js
  const indexResponse = await fetch('/index.json')
  const index = await indexResponse.json();

  const searchEngine = await create({
    schema: {
      title: 'string',
      content: 'string',
      url: 'string'
    },
    defaultLanguage: 'french', // i18n only
    components: {
      tokenizer: {
        stemmingFn: stemmer,
      }
    }
  });
  await insertBatch(searchEngine, index);

  const searchInput = document.getElementById('search-input');
  ['change', 'cut', 'input', 'paste', 'search'].forEach(type => searchInput.addEventListener(type, query));
  
  async function query(event) {
    const searchResponse = await search(searchEngine, {term: event.target.value, properties: '*'});
    document.getElementById('search-results').innerHTML = searchResponse
      .hits
      .map(i => `<a href="${i.document.url}" class="list-group-item list-group-item-action">${i.document.title}</a>`)
      .join('')
  }
```

Enjoy.

Thank you @mickaeltr !
