# Intl plugin for Sanity
[![npm version](https://img.shields.io/npm/v/sanity-plugin-intl-input.svg?style=flat)](https://www.npmjs.com/package/sanity-plugin-intl-input)

## Default solution
When you want to create translations in Sanity they suggest [following approach](https://www.sanity.io/docs/localization).  
This definitely works, but makes the UI very clunky as you get more fields that require translations.  

![Default Solution](https://raw.githubusercontent.com/LiamMartens/sanity-plugin-intl-input/master/doc/img/default-solution.gif)  

## With intl-plugin
With the intl plugin you will get a cleaner UI for creating translatable documents as the translation is managed across multiple fields and it is even possible to manage document wide translations.  

### Simple translated object field
![Intl Plugin Input](https://raw.githubusercontent.com/LiamMartens/sanity-plugin-intl-input/master/doc/img/intl-plugin.gif)  

### Document wide translation
![Intl Plugin Document Translation](https://raw.githubusercontent.com/LiamMartens/sanity-plugin-intl-input/master/doc/img/intl-plugin-document.gif)  

## How to install
1. Install the plugin using `npm`, `yarn` or the sanity cli
`yarn add sanity-plugin-intl-input`
`sanity install intl-input`
2. Add the plugin in the `sanity.json` of your project
```
{
  "root": true,
  "project": {
    "name": "..."
  },
  "plugins": [
    "...",
    "intl-input" // <--
  ]
}
```

3. The plugin has a custom desk structure to hide translated documents from the main view. You need to manually implement this as follows
Add the part in `sanity.json`
```
{
  ...
  "parts": [
    ...
    {
      "name": "part:@sanity/desk-tool/structure",
      "path": "./deskStructure.js"
    }
  ]
  ...
}
```

Implement the structure
```javascript
import * as Structure from 'sanity-plugin-intl-input/lib/structure';

// simpy re-exporting
export const getDefaultDocumentNode = Structure.getDefaultDocumentNode;
export default Structure.default;

// or manual implementation
export const getDefaultDocumentNode = (props) => {
    if (props.schemaType === 'myschema') {
        return S.document().views(Structure.getDocumentNodeViewsForSchemaType(props.schemaType));
    }
    return S.document();
};

export default () => {
  const items = Structure.getFilteredDocumentTypeListItems();
  return S.list()
      .id('__root__')
      .title('Content')
      .items(items);
};
```

## How to use
### Intl object input
Enable translations for your object type field which will contain all the translatable fields in your document type.  
```javascript
export default {
    type: 'document',
    name: '...',
    title: '...',
    fields: [{
        name: '...',
        title: '...',
        type: 'object',
        options: {
            i18n: true, // enables localization
            base: '', // (OPTIONAL) id of the base language (if not passed the first one is considered base language)
            languages: ['..', '..', ...], // <-- eg. ['en', 'nl']
            css: (classNames) => ``, // (OPTIONAL) function to apply additional CSS for theming purposes. The classNames argument is an object with the css module classnames.
            messages: { // (OPTIONAL) You can pass a messages object to override the default messsages shown
              loading: 'Loading languages...',
              missingTranslations: 'Missing translations message...',
            },
        },
        fields: []
    }]
}
```

### Intl for complete document
You can simply enable translations for your document type using the `i18n` parameter.
```javascript
export default {
    type: 'document',
    name: '...',
    title: '...',
    i18n: {
      base: '', // (OPTIONAL) id of the base language (if not passed the first one is considered base language)
      languages: ['..', '..', ...], // <-- eg. ['en', 'nl']
      messages: { // (OPTIONAL) You can pass a messages object to override the default messsages shown
        loading: 'Loading languages...',
        missing: 'Missing',
        draft: 'Draft',
        publishing: 'Publishing...',
        publish: 'Publish'
      },
      fieldName: { // (OPTIONAL) You can update the field name(s) the plugin injects
        lang: '__i18n_lang'
      }
    },
    fields: []
}
```

## Resulting data structure
### Intl object input
Your resulting object will not look much different than before apart from the fact that it will now group it's content within the langauge keys. eg:
```json
{
  "en": {
    "...": "..."
  },
  "fr": {
    "...": "..."
  }
}
```

### Document wide translations
Each document will have a new property called `__i18n_lang` (you can customize this by setting the `fieldNames.lang` property in the `i18n` settings object) which contains the name of the language of this document.
Secondly, the id will also contain a suffix as follows `xxx-xxx-xx__i18n_{lang}`.

#### GraphQL support
To make the field available in GraphQL you will need to add the language field in your own schemas so the sanity CLI can consume it.
Secondly, you will have to update the language field name to one that does not contain underscores.

In the document wide i18n setup:
```javascript
const schema = {
  i18n: {
    fieldNames: {
      lang: 'lang' // (for example)
    }
  }
};
```

In your own schema, add the following
```javascript
const schema = {
  fields: [
    {
      name: 'lang',
      type: 'string',
      hidden: true
    }
  ]
}
```

This way the GraphQL deployment will consider the `lang` property as part of your structure.

## Advanced languages
You can also pass a language objects or a GROQ query to the languages option for more advanced language handling.

### Language objects
If you pass an object with `name` and `title` to the languages array you can separate the data key and the visualized language name in the UI.
Eg:
```
languages: [
  { name: 'en_us', title: 'English (US)' },
  { name: 'en_gb', title: 'English (UK)' }
]
```

### GROQ query
It is also possible to pass a GROQ query to the languages option to dynamically fetch the available languages.
Eg:
```
languages: {
  query: '*[_type=="language"]{_id,name}',
  // these are the object paths to get out of the results to use for name and title
  // it is also possible to pass a simple string which will then be used for both title and name
  value: {
    title: 'name',
    name: '_id'
  }
}
```

