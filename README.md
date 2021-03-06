emoji-picker-element [![Build status](https://circleci.com/gh/nolanlawson/emoji-picker-element.svg?style=svg)](https://app.circleci.com/pipelines/gh/nolanlawson/emoji-picker-element)
====

![Screenshot of emoji-picker-element in light and dark modes](https://nolanwlawson.files.wordpress.com/2020/06/out.png)

```html
<emoji-picker></emoji-picker>
```

A lightweight emoji picker, distributed as a web component.

**Features:**

- Supports [Emoji v13.1](https://emojipedia.org/emoji-13.1/) (depending on OS) and custom emoji
- Uses IndexedDB, so it consumes [far less memory](https://nolanlawson.com/2020/06/28/introducing-emoji-picker-element-a-memory-efficient-emoji-picker-for-the-web/) than other emoji pickers
- [Small bundle size](https://bundlephobia.com/result?p=emoji-picker-element) (41kB minified, ~14.4kB gzipped)
- Renders native emoji only, no spritesheets
- Accessible by default
- Framework and bundler not required, just add a `<script>` tag and use it

**Table of contents:**

<!-- toc start -->

- [emoji-picker-element](#emoji-picker-element-)
  * [Usage](#usage)
    + [Examples](#examples)
  * [Styling](#styling)
    + [Size](#size)
    + [Dark mode](#dark-mode)
    + [CSS variables](#css-variables)
    + [Focus outline](#focus-outline)
    + [Custom styling](#custom-styling)
  * [JavaScript API](#javascript-api)
    + [Picker](#picker)
      - [i18n structure](#i18n-structure)
      - [Custom category order](#custom-category-order)
    + [Database](#database)
      - [Constructors](#constructors)
        * [constructor](#constructor)
      - [Accessors](#accessors)
        * [customEmoji](#customemoji)
      - [Methods](#methods)
        * [close](#close)
        * [delete](#delete)
        * [getEmojiByGroup](#getemojibygroup)
        * [getEmojiBySearchQuery](#getemojibysearchquery)
        * [getEmojiByShortcode](#getemojibyshortcode)
        * [getEmojiByUnicodeOrName](#getemojibyunicodeorname)
        * [getPreferredSkinTone](#getpreferredskintone)
        * [getTopFavoriteEmoji](#gettopfavoriteemoji)
        * [incrementFavoriteEmojiCount](#incrementfavoriteemojicount)
        * [ready](#ready)
        * [setPreferredSkinTone](#setpreferredskintone)
    + [Events](#events)
      - [`emoji-click`](#emoji-click)
      - [`skin-tone-change`](#skin-tone-change)
    + [Custom emoji](#custom-emoji)
    + [Tree-shaking](#tree-shaking)
    + [Within a Svelte project](#within-a-svelte-project)
  * [Data and offline](#data-and-offline)
    + [Data source and JSON format](#data-source-and-json-format)
    + [Shortcodes](#shortcodes)
    + [Cache performance](#cache-performance)
    + [emojibase-data compatibility (deprecated)](#emojibase-data-compatibility-deprecated)
    + [Trimming the emoji data (deprecated)](#trimming-the-emoji-data-deprecated)
    + [Offline-first](#offline-first)
    + [Environments without IndexedDB](#environments-without-indexeddb)
  * [Design decisions](#design-decisions)
    + [IndexedDB](#indexeddb)
    + [Native emoji](#native-emoji)
    + [JSON loading](#json-loading)
    + [Browser support](#browser-support)
  * [Contributing](#contributing)

<!-- toc end -->

## Usage

Via npm:

    npm install emoji-picker-element

```js
import 'emoji-picker-element';
```

Or as a `<script>` tag:

```html
<script type="module" src="https://cdn.jsdelivr.net/npm/emoji-picker-element@^1/index.js"></script>
```

Then use the HTML:

```html
<emoji-picker></emoji-picker>
```

And listen for `emoji-click` events:

```js
document.querySelector('emoji-picker')
  .addEventListener('emoji-click', event => console.log(event.detail));
```

This will log:

```json
{
  "emoji": {
    "annotation": "grinning face",
    "group": 0,
    "order": 1,
    "shortcodes": [ "grinning_face", "grinning" ],
    "tags": [ "face", "grin" ],
    "unicode": "😀",
    "version": 1,
    "emoticon": ":D"
  },
  "skinTone": 0,
  "unicode": "😀"
}
```

### Examples

- [Demo](https://nolanlawson.github.io/emoji-picker-element)
- [Button with tooltip/popover](https://bl.ocks.org/nolanlawson/781e7084e4c17acb921357489d51a5b0)

## Styling

`emoji-picker-element` uses [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM), so its inner styling cannot be (easily) changed with arbitrary CSS. Refer to the API below for style customization.

### Size

`emoji-picker-element` has a default size, but you can change it to whatever you want:

```css
emoji-picker {
  width: 400px;
  height: 300px;
}
```

For instance, to make it expand to fit whatever container you give it:

```css
emoji-picker {
  width: 100%;
  height: 100%;
}
```

### Dark mode

By default, `emoji-picker-element` will automatically switch to dark mode based on 
[`prefers-color-scheme`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme). 
Or you can add the class `dark` or `light` to force dark/light mode:

```html
<emoji-picker class="dark"></emoji-picker>
<emoji-picker class="light"></emoji-picker>
```

### CSS variables

Most colors and sizes can be styled with CSS variables. For example:

```css
emoji-picker {
  --num-columns: 6;
  --emoji-size: 3rem;
  --background: gray;
}
```

Here is a full list of options:

<!-- CSS variable options start -->

| Variable                     | Default    | Default (dark) | Description                                   |
| ---------------------------- | ---------- | -------------- | --------------------------------------------- |
| `--background`               | `#fff`     | `#222`         | Background of the entire `<emoji-picker>`     |
| `--border-color`             | `#e0e0e0`  | `#444`         |                                               |
| `--border-size`              | `1px`      |                | Width of border used in most of the picker    |
| `--button-active-background` | `#e6e6e6`  | `#555555`      | Background of an active button                |
| `--button-hover-background`  | `#d9d9d9`  | `#484848`      | Background of a hovered button                |
| `--category-font-color`      | `#111`     | `#efefef`      | Font color of custom emoji category headings  |
| `--category-font-size`       | `1rem`     |                | Font size of custom emoji category headings   |
| `--emoji-padding`            | `0.5rem`   |                |                                               |
| `--emoji-size`               | `1.375rem` |                |                                               |
| `--indicator-color`          | `#385ac1`  | `#5373ec`      | Color of the nav indicator                    |
| `--indicator-height`         | `3px`      |                | Height of the nav indicator                   |
| `--input-border-color`       | `#999`     | `#ccc`         |                                               |
| `--input-border-radius`      | `0.5rem`   |                |                                               |
| `--input-border-size`        | `1px`      |                |                                               |
| `--input-font-color`         | `#111`     | `#efefef`      |                                               |
| `--input-font-size`          | `1rem`     |                |                                               |
| `--input-line-height`        | `1.5`      |                |                                               |
| `--input-padding`            | `0.25rem`  |                |                                               |
| `--input-placeholder-color`  | `#999`     | `#ccc`         |                                               |
| `--num-columns`              | `8`        |                | How many columns to display in the emoji grid |
| `--outline-color`            | `#999`     | `#fff`         | Focus outline color                           |
| `--outline-size`             | `2px`      |                | Focus outline width                           |
| `--skintone-border-radius`   | `1rem`     |                | border radius of the skintone dropdown        |

<!-- CSS variable options end -->

### Focus outline

For accessibility reasons, `emoji-picker-element` displays a prominent focus ring. If you want to hide the focus ring for non-keyboard users (e.g. mouse and touch only), then use the [focus-visible](https://github.com/WICG/focus-visible) polyfill, e.g.:

```js
import 'focus-visible';

const picker = new Picker();
applyFocusVisiblePolyfill(picker.shadowRoot);
```

`emoji-picker-element` already ships with the proper CSS for both the `:focus-visible` standard and the polyfill.

### Custom styling

If you absolutely must go beyond the styling API above, you can do something like this:

```js
const style = document.createElement('style');
style.textContent = `/* custom shadow dom styles here */`
picker.shadowRoot.appendChild(style);
```

## JavaScript API

### Picker

Basic usage:

```js
import { Picker } from 'emoji-picker-element';
const picker = new Picker();
document.body.appendChild(picker);
```

The `new Picker(options)` constructor supports several options:

<!-- picker API start -->

Name | Type |
:------ | :------ |
`__namedParameters` | *PickerConstructorOptions* |



<!-- picker API end -->

For instance:

```js
const picker = new Picker({
  locale: 'fr',
  dataSource: '/fr-emoji.json'
})
```

These values can also be set at runtime:

```js
const picker = new Picker();
picker.dataSource = '/my-emoji.json';
```

Some values can also be set as declarative attributes:

```html
<emoji-picker
  locale="fr"
  data-source="/fr-emoji.json"
  skin-tone-emoji="✌"
></emoji-picker>
```

Note that complex properties like `i18n` or `customEmoji` are not supported as attributes, because the DOM only
supports string attributes, not complex objects.

#### i18n structure

Here is the default English `i18n` object (`"en"` locale):

<!-- i18n options start -->

```json
{
  "categories": {
    "custom": "Custom",
    "smileys-emotion": "Smileys and emoticons",
    "people-body": "People and body",
    "animals-nature": "Animals and nature",
    "food-drink": "Food and drink",
    "travel-places": "Travel and places",
    "activities": "Activities",
    "objects": "Objects",
    "symbols": "Symbols",
    "flags": "Flags"
  },
  "categoriesLabel": "Categories",
  "emojiUnsupportedMessage": "Your browser does not support color emoji.",
  "favoritesLabel": "Favorites",
  "loadingMessage": "Loading…",
  "networkErrorMessage": "Could not load emoji. Try refreshing.",
  "regionLabel": "Emoji picker",
  "searchDescription": "When search results are available, press up or down to select and enter to choose.",
  "searchLabel": "Search",
  "searchResultsLabel": "Search results",
  "skinToneDescription": "When expanded, press up or down to select and enter to choose.",
  "skinToneLabel": "Choose a skin tone (currently {skinTone})",
  "skinTones": [
    "Default",
    "Light",
    "Medium-Light",
    "Medium",
    "Medium-Dark",
    "Dark"
  ],
  "skinTonesLabel": "Skin tones"
}
```

<!-- i18n options end -->

Note that some of these strings are only visible to users of screen readers.
But you should still support them if you internationalize your app!

#### Custom category order

By default, custom categories are sorted alphabetically. To change this, pass in your own `customCategorySorting`:

```js
picker.customCategorySorting = (category1, category2) => { /* your sorting code */ };
```

This function should accept two strings and return a number.

Custom emoji with no category will pass in `undefined`. By default, these are shown first, with the label `"Custom"`
(determined by `i18n.categories.custom`).

### Database

You can work with the database API separately, which allows you to query emoji the same
way that the picker does:

```js
import { Database } from 'emoji-picker-element';

const database = new Database();
await database.getEmojiBySearchPrefix('elephant'); // [{unicode: "🐘", ...}]
```

Note that under the hood, IndexedDB data is partitioned based on the `locale`. So if you create two `Database`s with two different `locale`s, it will store twice as much data.

Full API:

<!-- database API start -->

#### Constructors

##### constructor

\+ **new default**(`__namedParameters?`: *PickerConstructorOptions*): *default*

###### Parameters:

Name | Type |
:------ | :------ |
`__namedParameters` | *PickerConstructorOptions* |

**Returns:** *default*

Defined in: src/types/picker.ts:9

#### Properties

##### ATTRIBUTE\_NODE

• `Readonly` **ATTRIBUTE\_NODE**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10866

___

##### CDATA\_SECTION\_NODE

• `Readonly` **CDATA\_SECTION\_NODE**: *number*

node is a CDATASection node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10870

___

##### COMMENT\_NODE

• `Readonly` **COMMENT\_NODE**: *number*

node is a Comment node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10874

___

##### DOCUMENT\_FRAGMENT\_NODE

• `Readonly` **DOCUMENT\_FRAGMENT\_NODE**: *number*

node is a DocumentFragment node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10878

___

##### DOCUMENT\_NODE

• `Readonly` **DOCUMENT\_NODE**: *number*

node is a document.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10882

___

##### DOCUMENT\_POSITION\_CONTAINED\_BY

• `Readonly` **DOCUMENT\_POSITION\_CONTAINED\_BY**: *number*

Set when other is a descendant of node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10886

___

##### DOCUMENT\_POSITION\_CONTAINS

• `Readonly` **DOCUMENT\_POSITION\_CONTAINS**: *number*

Set when other is an ancestor of node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10890

___

##### DOCUMENT\_POSITION\_DISCONNECTED

• `Readonly` **DOCUMENT\_POSITION\_DISCONNECTED**: *number*

Set when node and other are not in the same tree.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10894

___

##### DOCUMENT\_POSITION\_FOLLOWING

• `Readonly` **DOCUMENT\_POSITION\_FOLLOWING**: *number*

Set when other is following node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10898

___

##### DOCUMENT\_POSITION\_IMPLEMENTATION\_SPECIFIC

• `Readonly` **DOCUMENT\_POSITION\_IMPLEMENTATION\_SPECIFIC**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10899

___

##### DOCUMENT\_POSITION\_PRECEDING

• `Readonly` **DOCUMENT\_POSITION\_PRECEDING**: *number*

Set when other is preceding node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10903

___

##### DOCUMENT\_TYPE\_NODE

• `Readonly` **DOCUMENT\_TYPE\_NODE**: *number*

node is a doctype.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10907

___

##### ELEMENT\_NODE

• `Readonly` **ELEMENT\_NODE**: *number*

node is an element.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10911

___

##### ENTITY\_NODE

• `Readonly` **ENTITY\_NODE**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10912

___

##### ENTITY\_REFERENCE\_NODE

• `Readonly` **ENTITY\_REFERENCE\_NODE**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10913

___

##### NOTATION\_NODE

• `Readonly` **NOTATION\_NODE**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10914

___

##### PROCESSING\_INSTRUCTION\_NODE

• `Readonly` **PROCESSING\_INSTRUCTION\_NODE**: *number*

node is a ProcessingInstruction node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10918

___

##### TEXT\_NODE

• `Readonly` **TEXT\_NODE**: *number*

node is a Text node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10922

___

##### accessKey

• **accessKey**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6614

___

##### accessKeyLabel

• `Readonly` **accessKeyLabel**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6615

___

##### assignedSlot

• `Readonly` **assignedSlot**: HTMLSlotElement

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5080

___

##### attributes

• `Readonly` **attributes**: NamedNodeMap

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5081

___

##### autocapitalize

• **autocapitalize**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6616

___

##### autofocus

• **autofocus**: *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:8028

___

##### baseURI

• `Readonly` **baseURI**: *string*

Returns node's node document's document base URL.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10781

___

##### childElementCount

• `Readonly` **childElementCount**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11338

___

##### childNodes

• `Readonly` **childNodes**: *NodeListOf*<ChildNode\>

Returns the children.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10785

___

##### children

• `Readonly` **children**: HTMLCollection

Returns the child elements.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11342

___

##### classList

• `Readonly` **classList**: DOMTokenList

Allows for manipulation of element's class content attribute as a set of whitespace-separated tokens through a DOMTokenList object.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5085

___

##### className

• **className**: *string*

Returns the value of element's class content attribute. Can be set to change it.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5089

___

##### clientHeight

• `Readonly` **clientHeight**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5090

___

##### clientLeft

• `Readonly` **clientLeft**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5091

___

##### clientTop

• `Readonly` **clientTop**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5092

___

##### clientWidth

• `Readonly` **clientWidth**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5093

___

##### contentEditable

• **contentEditable**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5245

___

##### customCategorySorting

• `Optional` **customCategorySorting**: (`a`: *string*, `b`: *string*) => *number*

###### Type declaration:

▸ (`a`: *string*, `b`: *string*): *number*

###### Parameters:

Name | Type |
:------ | :------ |
`a` | *string* |
`b` | *string* |

**Returns:** *number*

Defined in: src/types/picker.ts:9

Defined in: src/types/picker.ts:9

___

##### customEmoji

• `Optional` **customEmoji**: *CustomEmoji*[]

Defined in: src/types/picker.ts:8

___

##### dataSource

• **dataSource**: *string*

Defined in: src/types/picker.ts:4

___

##### dataset

• `Readonly` **dataset**: DOMStringMap

Defined in: node_modules/typescript/lib/lib.dom.d.ts:8029

___

##### dir

• **dir**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6617

___

##### draggable

• **draggable**: *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6618

___

##### enterKeyHint

• **enterKeyHint**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5246

___

##### firstChild

• `Readonly` **firstChild**: ChildNode

Returns the first child.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10789

___

##### firstElementChild

• `Readonly` **firstElementChild**: Element

Returns the first child that is an element, and null otherwise.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11346

___

##### hidden

• **hidden**: *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6619

___

##### i18n

• **i18n**: *I18n*

Defined in: src/types/picker.ts:6

___

##### id

• **id**: *string*

Returns the value of element's id content attribute. Can be set to change it.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5097

___

##### innerHTML

• **innerHTML**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:9611

___

##### innerText

• **innerText**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6620

___

##### inputMode

• **inputMode**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5247

___

##### isConnected

• `Readonly` **isConnected**: *boolean*

Returns true if node is connected and false otherwise.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10793

___

##### isContentEditable

• `Readonly` **isContentEditable**: *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5248

___

##### lang

• **lang**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6621

___

##### lastChild

• `Readonly` **lastChild**: ChildNode

Returns the last child.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10797

___

##### lastElementChild

• `Readonly` **lastElementChild**: Element

Returns the last child that is an element, and null otherwise.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11350

___

##### localName

• `Readonly` **localName**: *string*

Returns the local name.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5101

___

##### locale

• **locale**: *string*

Defined in: src/types/picker.ts:5

___

##### namespaceURI

• `Readonly` **namespaceURI**: *string*

Returns the namespace.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5105

___

##### nextElementSibling

• `Readonly` **nextElementSibling**: Element

Returns the first following sibling that is an element, and null otherwise.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11068

___

##### nextSibling

• `Readonly` **nextSibling**: ChildNode

Returns the next sibling.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10803

___

##### nodeName

• `Readonly` **nodeName**: *string*

Returns a string appropriate for the type of node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10807

___

##### nodeType

• `Readonly` **nodeType**: *number*

Returns the type of node.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10811

___

##### nodeValue

• **nodeValue**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10812

___

##### nonce

• `Optional` **nonce**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:8030

___

##### offsetHeight

• `Readonly` **offsetHeight**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6622

___

##### offsetLeft

• `Readonly` **offsetLeft**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6623

___

##### offsetParent

• `Readonly` **offsetParent**: Element

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6624

___

##### offsetTop

• `Readonly` **offsetTop**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6625

___

##### offsetWidth

• `Readonly` **offsetWidth**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6626

___

##### onabort

• **onabort**: (`ev`: UIEvent) => *any*

Fires when the user aborts the download.

**`param`** The event.

###### Type declaration:

▸ (`ev`: UIEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | UIEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5804

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5804

___

##### onanimationcancel

• **onanimationcancel**: (`ev`: AnimationEvent) => *any*

###### Type declaration:

▸ (`ev`: AnimationEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | AnimationEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5805

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5805

___

##### onanimationend

• **onanimationend**: (`ev`: AnimationEvent) => *any*

###### Type declaration:

▸ (`ev`: AnimationEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | AnimationEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5806

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5806

___

##### onanimationiteration

• **onanimationiteration**: (`ev`: AnimationEvent) => *any*

###### Type declaration:

▸ (`ev`: AnimationEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | AnimationEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5807

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5807

___

##### onanimationstart

• **onanimationstart**: (`ev`: AnimationEvent) => *any*

###### Type declaration:

▸ (`ev`: AnimationEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | AnimationEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5808

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5808

___

##### onauxclick

• **onauxclick**: (`ev`: MouseEvent) => *any*

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5809

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5809

___

##### onblur

• **onblur**: (`ev`: FocusEvent) => *any*

Fires when the object loses the input focus.

**`param`** The focus event.

###### Type declaration:

▸ (`ev`: FocusEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | FocusEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5814

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5814

___

##### oncancel

• **oncancel**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5815

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5815

___

##### oncanplay

• **oncanplay**: (`ev`: Event) => *any*

Occurs when playback is possible, but would require further buffering.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5820

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5820

___

##### oncanplaythrough

• **oncanplaythrough**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5821

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5821

___

##### onchange

• **onchange**: (`ev`: Event) => *any*

Fires when the contents of the object or selection have changed.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5826

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5826

___

##### onclick

• **onclick**: (`ev`: MouseEvent) => *any*

Fires when the user clicks the left mouse button on the object

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5831

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5831

___

##### onclose

• **onclose**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5832

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5832

___

##### oncontextmenu

• **oncontextmenu**: (`ev`: MouseEvent) => *any*

Fires when the user clicks the right mouse button in the client area, opening the context menu.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5837

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5837

___

##### oncopy

• **oncopy**: (`ev`: ClipboardEvent) => *any*

###### Type declaration:

▸ (`ev`: ClipboardEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | ClipboardEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:4879

Defined in: node_modules/typescript/lib/lib.dom.d.ts:4879

___

##### oncuechange

• **oncuechange**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5838

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5838

___

##### oncut

• **oncut**: (`ev`: ClipboardEvent) => *any*

###### Type declaration:

▸ (`ev`: ClipboardEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | ClipboardEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:4880

Defined in: node_modules/typescript/lib/lib.dom.d.ts:4880

___

##### ondblclick

• **ondblclick**: (`ev`: MouseEvent) => *any*

Fires when the user double-clicks the object.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5843

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5843

___

##### ondrag

• **ondrag**: (`ev`: DragEvent) => *any*

Fires on the source object continuously during a drag operation.

**`param`** The event.

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5848

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5848

___

##### ondragend

• **ondragend**: (`ev`: DragEvent) => *any*

Fires on the source object when the user releases the mouse at the close of a drag operation.

**`param`** The event.

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5853

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5853

___

##### ondragenter

• **ondragenter**: (`ev`: DragEvent) => *any*

Fires on the target element when the user drags the object to a valid drop target.

**`param`** The drag event.

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5858

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5858

___

##### ondragexit

• **ondragexit**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5859

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5859

___

##### ondragleave

• **ondragleave**: (`ev`: DragEvent) => *any*

Fires on the target object when the user moves the mouse out of a valid drop target during a drag operation.

**`param`** The drag event.

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5864

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5864

___

##### ondragover

• **ondragover**: (`ev`: DragEvent) => *any*

Fires on the target element continuously while the user drags the object over a valid drop target.

**`param`** The event.

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5869

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5869

___

##### ondragstart

• **ondragstart**: (`ev`: DragEvent) => *any*

Fires on the source object when the user starts to drag a text selection or selected object.

**`param`** The event.

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5874

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5874

___

##### ondrop

• **ondrop**: (`ev`: DragEvent) => *any*

###### Type declaration:

▸ (`ev`: DragEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | DragEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5875

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5875

___

##### ondurationchange

• **ondurationchange**: (`ev`: Event) => *any*

Occurs when the duration attribute is updated.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5880

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5880

___

##### onemptied

• **onemptied**: (`ev`: Event) => *any*

Occurs when the media element is reset to its initial state.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5885

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5885

___

##### onended

• **onended**: (`ev`: Event) => *any*

Occurs when the end of playback is reached.

**`param`** The event

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5890

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5890

___

##### onerror

• **onerror**: OnErrorEventHandlerNonNull

Fires when an error occurs during object loading.

**`param`** The event.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5895

___

##### onfocus

• **onfocus**: (`ev`: FocusEvent) => *any*

Fires when the object receives focus.

**`param`** The event.

###### Type declaration:

▸ (`ev`: FocusEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | FocusEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5900

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5900

___

##### onfullscreenchange

• **onfullscreenchange**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5106

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5106

___

##### onfullscreenerror

• **onfullscreenerror**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5107

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5107

___

##### ongotpointercapture

• **ongotpointercapture**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5901

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5901

___

##### oninput

• **oninput**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5902

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5902

___

##### oninvalid

• **oninvalid**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5903

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5903

___

##### onkeydown

• **onkeydown**: (`ev`: KeyboardEvent) => *any*

Fires when the user presses a key.

**`param`** The keyboard event

###### Type declaration:

▸ (`ev`: KeyboardEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | KeyboardEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5908

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5908

___

##### onkeypress

• **onkeypress**: (`ev`: KeyboardEvent) => *any*

Fires when the user presses an alphanumeric key.

**`param`** The event.

###### Type declaration:

▸ (`ev`: KeyboardEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | KeyboardEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5913

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5913

___

##### onkeyup

• **onkeyup**: (`ev`: KeyboardEvent) => *any*

Fires when the user releases a key.

**`param`** The keyboard event

###### Type declaration:

▸ (`ev`: KeyboardEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | KeyboardEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5918

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5918

___

##### onload

• **onload**: (`ev`: Event) => *any*

Fires immediately after the browser loads the object.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5923

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5923

___

##### onloadeddata

• **onloadeddata**: (`ev`: Event) => *any*

Occurs when media data is loaded at the current playback position.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5928

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5928

___

##### onloadedmetadata

• **onloadedmetadata**: (`ev`: Event) => *any*

Occurs when the duration and dimensions of the media have been determined.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5933

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5933

___

##### onloadstart

• **onloadstart**: (`ev`: Event) => *any*

Occurs when Internet Explorer begins looking for media data.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5938

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5938

___

##### onlostpointercapture

• **onlostpointercapture**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5939

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5939

___

##### onmousedown

• **onmousedown**: (`ev`: MouseEvent) => *any*

Fires when the user clicks the object with either mouse button.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5944

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5944

___

##### onmouseenter

• **onmouseenter**: (`ev`: MouseEvent) => *any*

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5945

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5945

___

##### onmouseleave

• **onmouseleave**: (`ev`: MouseEvent) => *any*

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5946

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5946

___

##### onmousemove

• **onmousemove**: (`ev`: MouseEvent) => *any*

Fires when the user moves the mouse over the object.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5951

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5951

___

##### onmouseout

• **onmouseout**: (`ev`: MouseEvent) => *any*

Fires when the user moves the mouse pointer outside the boundaries of the object.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5956

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5956

___

##### onmouseover

• **onmouseover**: (`ev`: MouseEvent) => *any*

Fires when the user moves the mouse pointer into the object.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5961

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5961

___

##### onmouseup

• **onmouseup**: (`ev`: MouseEvent) => *any*

Fires when the user releases a mouse button while the mouse is over the object.

**`param`** The mouse event.

###### Type declaration:

▸ (`ev`: MouseEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | MouseEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5966

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5966

___

##### onpaste

• **onpaste**: (`ev`: ClipboardEvent) => *any*

###### Type declaration:

▸ (`ev`: ClipboardEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | ClipboardEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:4881

Defined in: node_modules/typescript/lib/lib.dom.d.ts:4881

___

##### onpause

• **onpause**: (`ev`: Event) => *any*

Occurs when playback is paused.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5971

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5971

___

##### onplay

• **onplay**: (`ev`: Event) => *any*

Occurs when the play method is requested.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5976

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5976

___

##### onplaying

• **onplaying**: (`ev`: Event) => *any*

Occurs when the audio or video has started playing.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5981

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5981

___

##### onpointercancel

• **onpointercancel**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5982

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5982

___

##### onpointerdown

• **onpointerdown**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5983

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5983

___

##### onpointerenter

• **onpointerenter**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5984

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5984

___

##### onpointerleave

• **onpointerleave**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5985

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5985

___

##### onpointermove

• **onpointermove**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5986

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5986

___

##### onpointerout

• **onpointerout**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5987

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5987

___

##### onpointerover

• **onpointerover**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5988

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5988

___

##### onpointerup

• **onpointerup**: (`ev`: PointerEvent) => *any*

###### Type declaration:

▸ (`ev`: PointerEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | PointerEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5989

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5989

___

##### onprogress

• **onprogress**: (`ev`: *ProgressEvent*<EventTarget\>) => *any*

Occurs to indicate progress while downloading media data.

**`param`** The event.

###### Type declaration:

▸ (`ev`: *ProgressEvent*<EventTarget\>): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | *ProgressEvent*<EventTarget\> |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5994

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5994

___

##### onratechange

• **onratechange**: (`ev`: Event) => *any*

Occurs when the playback rate is increased or decreased.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5999

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5999

___

##### onreset

• **onreset**: (`ev`: Event) => *any*

Fires when the user resets a form.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6004

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6004

___

##### onresize

• **onresize**: (`ev`: UIEvent) => *any*

###### Type declaration:

▸ (`ev`: UIEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | UIEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6005

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6005

___

##### onscroll

• **onscroll**: (`ev`: Event) => *any*

Fires when the user repositions the scroll box in the scroll bar on the object.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6010

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6010

___

##### onsecuritypolicyviolation

• **onsecuritypolicyviolation**: (`ev`: SecurityPolicyViolationEvent) => *any*

###### Type declaration:

▸ (`ev`: SecurityPolicyViolationEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | SecurityPolicyViolationEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6011

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6011

___

##### onseeked

• **onseeked**: (`ev`: Event) => *any*

Occurs when the seek operation ends.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6016

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6016

___

##### onseeking

• **onseeking**: (`ev`: Event) => *any*

Occurs when the current playback position is moved.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6021

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6021

___

##### onselect

• **onselect**: (`ev`: Event) => *any*

Fires when the current selection changes.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6026

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6026

___

##### onselectionchange

• **onselectionchange**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6027

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6027

___

##### onselectstart

• **onselectstart**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6028

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6028

___

##### onstalled

• **onstalled**: (`ev`: Event) => *any*

Occurs when the download has stopped.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6033

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6033

___

##### onsubmit

• **onsubmit**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6034

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6034

___

##### onsuspend

• **onsuspend**: (`ev`: Event) => *any*

Occurs if the load operation has been intentionally halted.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6039

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6039

___

##### ontimeupdate

• **ontimeupdate**: (`ev`: Event) => *any*

Occurs to indicate the current playback position.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6044

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6044

___

##### ontoggle

• **ontoggle**: (`ev`: Event) => *any*

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6045

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6045

___

##### ontouchcancel

• `Optional` **ontouchcancel**: (`ev`: TouchEvent) => *any*

###### Type declaration:

▸ (`ev`: TouchEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TouchEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6046

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6046

___

##### ontouchend

• `Optional` **ontouchend**: (`ev`: TouchEvent) => *any*

###### Type declaration:

▸ (`ev`: TouchEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TouchEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6047

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6047

___

##### ontouchmove

• `Optional` **ontouchmove**: (`ev`: TouchEvent) => *any*

###### Type declaration:

▸ (`ev`: TouchEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TouchEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6048

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6048

___

##### ontouchstart

• `Optional` **ontouchstart**: (`ev`: TouchEvent) => *any*

###### Type declaration:

▸ (`ev`: TouchEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TouchEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6049

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6049

___

##### ontransitioncancel

• **ontransitioncancel**: (`ev`: TransitionEvent) => *any*

###### Type declaration:

▸ (`ev`: TransitionEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TransitionEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6050

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6050

___

##### ontransitionend

• **ontransitionend**: (`ev`: TransitionEvent) => *any*

###### Type declaration:

▸ (`ev`: TransitionEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TransitionEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6051

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6051

___

##### ontransitionrun

• **ontransitionrun**: (`ev`: TransitionEvent) => *any*

###### Type declaration:

▸ (`ev`: TransitionEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TransitionEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6052

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6052

___

##### ontransitionstart

• **ontransitionstart**: (`ev`: TransitionEvent) => *any*

###### Type declaration:

▸ (`ev`: TransitionEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | TransitionEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6053

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6053

___

##### onvolumechange

• **onvolumechange**: (`ev`: Event) => *any*

Occurs when the volume is changed, or playback is muted or unmuted.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6058

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6058

___

##### onwaiting

• **onwaiting**: (`ev`: Event) => *any*

Occurs when playback stops because the next frame of a video resource is not available.

**`param`** The event.

###### Type declaration:

▸ (`ev`: Event): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | Event |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6063

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6063

___

##### onwheel

• **onwheel**: (`ev`: WheelEvent) => *any*

###### Type declaration:

▸ (`ev`: WheelEvent): *any*

###### Parameters:

Name | Type |
:------ | :------ |
`ev` | WheelEvent |

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6064

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6064

___

##### outerHTML

• **outerHTML**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5108

___

##### ownerDocument

• `Readonly` **ownerDocument**: Document

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5109

___

##### parentElement

• `Readonly` **parentElement**: HTMLElement

Returns the parent element.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10820

___

##### parentNode

• `Readonly` **parentNode**: Node & ParentNode

Returns the parent.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10824

___

##### prefix

• `Readonly` **prefix**: *string*

Returns the namespace prefix.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5113

___

##### previousElementSibling

• `Readonly` **previousElementSibling**: Element

Returns the first preceding sibling that is an element, and null otherwise.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11072

___

##### previousSibling

• `Readonly` **previousSibling**: ChildNode

Returns the previous sibling.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10828

___

##### scrollHeight

• `Readonly` **scrollHeight**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5114

___

##### scrollLeft

• **scrollLeft**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5115

___

##### scrollTop

• **scrollTop**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5116

___

##### scrollWidth

• `Readonly` **scrollWidth**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5117

___

##### shadowRoot

• `Readonly` **shadowRoot**: ShadowRoot

Returns element's shadow root, if any, and if shadow root's mode is "open", and null otherwise.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5121

___

##### skinToneEmoji

• **skinToneEmoji**: *string*

Defined in: src/types/picker.ts:7

___

##### slot

• **slot**: *string*

Returns the value of element's slot content attribute. Can be set to change it.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5125

___

##### spellcheck

• **spellcheck**: *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6627

___

##### style

• `Readonly` **style**: CSSStyleDeclaration

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5241

___

##### tabIndex

• **tabIndex**: *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:8031

___

##### tagName

• `Readonly` **tagName**: *string*

Returns the HTML-uppercased qualified name.

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5129

___

##### textContent

• **textContent**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10829

___

##### title

• **title**: *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6628

___

##### translate

• **translate**: *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6629

#### Methods

##### addEventListener

▸ **addEventListener**<K\>(`type`: K, `listener`: (`ev`: *EmojiPickerEventMap*[K]) => *any*, `options?`: *boolean* \| AddEventListenerOptions): *void*

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *emoji-click* \| *skin-tone-change* |

###### Parameters:

Name | Type |
:------ | :------ |
`type` | K |
`listener` | (`ev`: *EmojiPickerEventMap*[K]) => *any* |
`options?` | *boolean* \| AddEventListenerOptions |

**Returns:** *void*

Defined in: src/types/picker.ts:31

___

##### after

▸ **after**(...`nodes`: (*string* \| Node)[]): *void*

Inserts nodes just after node, while replacing strings in nodes with equivalent Text nodes.

Throws a "HierarchyRequestError" DOMException if the constraints of the node tree are violated.

###### Parameters:

Name | Type |
:------ | :------ |
`...nodes` | (*string* \| Node)[] |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:3498

___

##### animate

▸ **animate**(`keyframes`: Keyframe[] \| PropertyIndexedKeyframes, `options?`: *number* \| KeyframeAnimationOptions): Animation

###### Parameters:

Name | Type |
:------ | :------ |
`keyframes` | Keyframe[] \| PropertyIndexedKeyframes |
`options?` | *number* \| KeyframeAnimationOptions |

**Returns:** Animation

Defined in: node_modules/typescript/lib/lib.dom.d.ts:2036

___

##### append

▸ **append**(...`nodes`: (*string* \| Node)[]): *void*

Inserts nodes after the last child of node, while replacing strings in nodes with equivalent Text nodes.

Throws a "HierarchyRequestError" DOMException if the constraints of the node tree are violated.

###### Parameters:

Name | Type |
:------ | :------ |
`...nodes` | (*string* \| Node)[] |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11356

___

##### appendChild

▸ **appendChild**<T\>(`newChild`: T): T

###### Type parameters:

Name | Type |
:------ | :------ |
`T` | Node |

###### Parameters:

Name | Type |
:------ | :------ |
`newChild` | T |

**Returns:** T

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10830

___

##### attachShadow

▸ **attachShadow**(`init`: ShadowRootInit): ShadowRoot

Creates a shadow root for element and returns it.

###### Parameters:

Name | Type |
:------ | :------ |
`init` | ShadowRootInit |

**Returns:** ShadowRoot

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5133

___

##### before

▸ **before**(...`nodes`: (*string* \| Node)[]): *void*

Inserts nodes just before node, while replacing strings in nodes with equivalent Text nodes.

Throws a "HierarchyRequestError" DOMException if the constraints of the node tree are violated.

###### Parameters:

Name | Type |
:------ | :------ |
`...nodes` | (*string* \| Node)[] |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:3504

___

##### blur

▸ **blur**(): *void*

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:8032

___

##### click

▸ **click**(): *void*

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:6630

___

##### cloneNode

▸ **cloneNode**(`deep?`: *boolean*): Node

Returns a copy of node. If deep is true, the copy also includes the node's descendants.

###### Parameters:

Name | Type |
:------ | :------ |
`deep?` | *boolean* |

**Returns:** Node

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10834

___

##### closest

▸ **closest**<K\>(`selector`: K): HTMLElementTagNameMap[K]

Returns the first (starting at element) inclusive ancestor that matches selectors, and null otherwise.

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *object* \| *a* \| *abbr* \| *address* \| *applet* \| *area* \| *article* \| *aside* \| *audio* \| *b* \| *base* \| *basefont* \| *bdi* \| *bdo* \| *blockquote* \| *body* \| *br* \| *button* \| *canvas* \| *caption* \| *cite* \| *code* \| *col* \| *colgroup* \| *data* \| *datalist* \| *dd* \| *del* \| *details* \| *dfn* \| *dialog* \| *dir* \| *div* \| *dl* \| *dt* \| *em* \| *embed* \| *fieldset* \| *figcaption* \| *figure* \| *font* \| *footer* \| *form* \| *frame* \| *frameset* \| *h1* \| *h2* \| *h3* \| *h4* \| *h5* \| *h6* \| *head* \| *header* \| *hgroup* \| *hr* \| *html* \| *i* \| *iframe* \| *img* \| *input* \| *ins* \| *kbd* \| *label* \| *legend* \| *li* \| *link* \| *main* \| *map* \| *mark* \| *marquee* \| *menu* \| *meta* \| *meter* \| *nav* \| *noscript* \| *ol* \| *optgroup* \| *option* \| *output* \| *p* \| *param* \| *picture* \| *pre* \| *progress* \| *q* \| *rp* \| *rt* \| *ruby* \| *s* \| *samp* \| *script* \| *section* \| *select* \| *slot* \| *small* \| *source* \| *span* \| *strong* \| *style* \| *sub* \| *summary* \| *sup* \| *table* \| *tbody* \| *td* \| *template* \| *textarea* \| *tfoot* \| *th* \| *thead* \| *time* \| *title* \| *tr* \| *track* \| *u* \| *ul* \| *var* \| *video* \| *wbr* \| *emoji-picker* |

###### Parameters:

Name | Type |
:------ | :------ |
`selector` | K |

**Returns:** HTMLElementTagNameMap[K]

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5137

▸ **closest**<K\>(`selector`: K): SVGElementTagNameMap[K]

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *symbol* \| *a* \| *script* \| *style* \| *title* \| *circle* \| *clipPath* \| *defs* \| *desc* \| *ellipse* \| *feBlend* \| *feColorMatrix* \| *feComponentTransfer* \| *feComposite* \| *feConvolveMatrix* \| *feDiffuseLighting* \| *feDisplacementMap* \| *feDistantLight* \| *feFlood* \| *feFuncA* \| *feFuncB* \| *feFuncG* \| *feFuncR* \| *feGaussianBlur* \| *feImage* \| *feMerge* \| *feMergeNode* \| *feMorphology* \| *feOffset* \| *fePointLight* \| *feSpecularLighting* \| *feSpotLight* \| *feTile* \| *feTurbulence* \| *filter* \| *foreignObject* \| *g* \| *image* \| *line* \| *linearGradient* \| *marker* \| *mask* \| *metadata* \| *path* \| *pattern* \| *polygon* \| *polyline* \| *radialGradient* \| *rect* \| *stop* \| *svg* \| *switch* \| *text* \| *textPath* \| *tspan* \| *use* \| *view* |

###### Parameters:

Name | Type |
:------ | :------ |
`selector` | K |

**Returns:** SVGElementTagNameMap[K]

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5138

▸ **closest**<E\>(`selector`: *string*): E

###### Type parameters:

Name | Type | Default |
:------ | :------ | :------ |
`E` | Element | Element |

###### Parameters:

Name | Type |
:------ | :------ |
`selector` | *string* |

**Returns:** E

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5139

___

##### compareDocumentPosition

▸ **compareDocumentPosition**(`other`: Node): *number*

Returns a bitmask indicating the position of other relative to node.

###### Parameters:

Name | Type |
:------ | :------ |
`other` | Node |

**Returns:** *number*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10838

___

##### contains

▸ **contains**(`other`: Node): *boolean*

Returns true if other is an inclusive descendant of node, and false otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`other` | Node |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10842

___

##### dispatchEvent

▸ **dispatchEvent**(`event`: Event): *boolean*

Dispatches a synthetic event event to target and returns true if either event's cancelable attribute value is false or its preventDefault() method was not invoked, and false otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`event` | Event |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5409

___

##### focus

▸ **focus**(`options?`: FocusOptions): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`options?` | FocusOptions |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:8033

___

##### getAnimations

▸ **getAnimations**(): Animation[]

**Returns:** Animation[]

Defined in: node_modules/typescript/lib/lib.dom.d.ts:2037

___

##### getAttribute

▸ **getAttribute**(`qualifiedName`: *string*): *string*

Returns element's first attribute whose qualified name is qualifiedName, and null if there is no such attribute otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |

**Returns:** *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5143

___

##### getAttributeNS

▸ **getAttributeNS**(`namespace`: *string*, `localName`: *string*): *string*

Returns element's attribute whose namespace is namespace and local name is localName, and null if there is no such attribute otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |
`localName` | *string* |

**Returns:** *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5147

___

##### getAttributeNames

▸ **getAttributeNames**(): *string*[]

Returns the qualified names of all element's attributes. Can contain duplicates.

**Returns:** *string*[]

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5151

___

##### getAttributeNode

▸ **getAttributeNode**(`qualifiedName`: *string*): Attr

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |

**Returns:** Attr

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5152

___

##### getAttributeNodeNS

▸ **getAttributeNodeNS**(`namespace`: *string*, `localName`: *string*): Attr

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |
`localName` | *string* |

**Returns:** Attr

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5153

___

##### getBoundingClientRect

▸ **getBoundingClientRect**(): DOMRect

**Returns:** DOMRect

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5154

___

##### getClientRects

▸ **getClientRects**(): DOMRectList

**Returns:** DOMRectList

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5155

___

##### getElementsByClassName

▸ **getElementsByClassName**(`classNames`: *string*): *HTMLCollectionOf*<Element\>

Returns a HTMLCollection of the elements in the object on which the method was invoked (a document or an element) that have all the classes given by classNames. The classNames argument is interpreted as a space-separated list of classes.

###### Parameters:

Name | Type |
:------ | :------ |
`classNames` | *string* |

**Returns:** *HTMLCollectionOf*<Element\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5159

___

##### getElementsByTagName

▸ **getElementsByTagName**<K\>(`qualifiedName`: K): *HTMLCollectionOf*<HTMLElementTagNameMap[K]\>

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *object* \| *a* \| *abbr* \| *address* \| *applet* \| *area* \| *article* \| *aside* \| *audio* \| *b* \| *base* \| *basefont* \| *bdi* \| *bdo* \| *blockquote* \| *body* \| *br* \| *button* \| *canvas* \| *caption* \| *cite* \| *code* \| *col* \| *colgroup* \| *data* \| *datalist* \| *dd* \| *del* \| *details* \| *dfn* \| *dialog* \| *dir* \| *div* \| *dl* \| *dt* \| *em* \| *embed* \| *fieldset* \| *figcaption* \| *figure* \| *font* \| *footer* \| *form* \| *frame* \| *frameset* \| *h1* \| *h2* \| *h3* \| *h4* \| *h5* \| *h6* \| *head* \| *header* \| *hgroup* \| *hr* \| *html* \| *i* \| *iframe* \| *img* \| *input* \| *ins* \| *kbd* \| *label* \| *legend* \| *li* \| *link* \| *main* \| *map* \| *mark* \| *marquee* \| *menu* \| *meta* \| *meter* \| *nav* \| *noscript* \| *ol* \| *optgroup* \| *option* \| *output* \| *p* \| *param* \| *picture* \| *pre* \| *progress* \| *q* \| *rp* \| *rt* \| *ruby* \| *s* \| *samp* \| *script* \| *section* \| *select* \| *slot* \| *small* \| *source* \| *span* \| *strong* \| *style* \| *sub* \| *summary* \| *sup* \| *table* \| *tbody* \| *td* \| *template* \| *textarea* \| *tfoot* \| *th* \| *thead* \| *time* \| *title* \| *tr* \| *track* \| *u* \| *ul* \| *var* \| *video* \| *wbr* \| *emoji-picker* |

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | K |

**Returns:** *HTMLCollectionOf*<HTMLElementTagNameMap[K]\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5160

▸ **getElementsByTagName**<K\>(`qualifiedName`: K): *HTMLCollectionOf*<SVGElementTagNameMap[K]\>

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *symbol* \| *a* \| *script* \| *style* \| *title* \| *circle* \| *clipPath* \| *defs* \| *desc* \| *ellipse* \| *feBlend* \| *feColorMatrix* \| *feComponentTransfer* \| *feComposite* \| *feConvolveMatrix* \| *feDiffuseLighting* \| *feDisplacementMap* \| *feDistantLight* \| *feFlood* \| *feFuncA* \| *feFuncB* \| *feFuncG* \| *feFuncR* \| *feGaussianBlur* \| *feImage* \| *feMerge* \| *feMergeNode* \| *feMorphology* \| *feOffset* \| *fePointLight* \| *feSpecularLighting* \| *feSpotLight* \| *feTile* \| *feTurbulence* \| *filter* \| *foreignObject* \| *g* \| *image* \| *line* \| *linearGradient* \| *marker* \| *mask* \| *metadata* \| *path* \| *pattern* \| *polygon* \| *polyline* \| *radialGradient* \| *rect* \| *stop* \| *svg* \| *switch* \| *text* \| *textPath* \| *tspan* \| *use* \| *view* |

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | K |

**Returns:** *HTMLCollectionOf*<SVGElementTagNameMap[K]\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5161

▸ **getElementsByTagName**(`qualifiedName`: *string*): *HTMLCollectionOf*<Element\>

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |

**Returns:** *HTMLCollectionOf*<Element\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5162

___

##### getElementsByTagNameNS

▸ **getElementsByTagNameNS**(`namespaceURI`: *http://www.w3.org/1999/xhtml*, `localName`: *string*): *HTMLCollectionOf*<HTMLElement\>

###### Parameters:

Name | Type |
:------ | :------ |
`namespaceURI` | *http://www.w3.org/1999/xhtml* |
`localName` | *string* |

**Returns:** *HTMLCollectionOf*<HTMLElement\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5163

▸ **getElementsByTagNameNS**(`namespaceURI`: *http://www.w3.org/2000/svg*, `localName`: *string*): *HTMLCollectionOf*<SVGElement\>

###### Parameters:

Name | Type |
:------ | :------ |
`namespaceURI` | *http://www.w3.org/2000/svg* |
`localName` | *string* |

**Returns:** *HTMLCollectionOf*<SVGElement\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5164

▸ **getElementsByTagNameNS**(`namespaceURI`: *string*, `localName`: *string*): *HTMLCollectionOf*<Element\>

###### Parameters:

Name | Type |
:------ | :------ |
`namespaceURI` | *string* |
`localName` | *string* |

**Returns:** *HTMLCollectionOf*<Element\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5165

___

##### getRootNode

▸ **getRootNode**(`options?`: GetRootNodeOptions): Node

Returns node's root.

###### Parameters:

Name | Type |
:------ | :------ |
`options?` | GetRootNodeOptions |

**Returns:** Node

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10846

___

##### hasAttribute

▸ **hasAttribute**(`qualifiedName`: *string*): *boolean*

Returns true if element has an attribute whose qualified name is qualifiedName, and false otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5169

___

##### hasAttributeNS

▸ **hasAttributeNS**(`namespace`: *string*, `localName`: *string*): *boolean*

Returns true if element has an attribute whose namespace is namespace and local name is localName.

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |
`localName` | *string* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5173

___

##### hasAttributes

▸ **hasAttributes**(): *boolean*

Returns true if element has attributes, and false otherwise.

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5177

___

##### hasChildNodes

▸ **hasChildNodes**(): *boolean*

Returns whether node has children.

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10850

___

##### hasPointerCapture

▸ **hasPointerCapture**(`pointerId`: *number*): *boolean*

###### Parameters:

Name | Type |
:------ | :------ |
`pointerId` | *number* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5178

___

##### insertAdjacentElement

▸ **insertAdjacentElement**(`position`: InsertPosition, `insertedElement`: Element): Element

###### Parameters:

Name | Type |
:------ | :------ |
`position` | InsertPosition |
`insertedElement` | Element |

**Returns:** Element

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5179

___

##### insertAdjacentHTML

▸ **insertAdjacentHTML**(`where`: InsertPosition, `html`: *string*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`where` | InsertPosition |
`html` | *string* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5180

___

##### insertAdjacentText

▸ **insertAdjacentText**(`where`: InsertPosition, `text`: *string*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`where` | InsertPosition |
`text` | *string* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5181

___

##### insertBefore

▸ **insertBefore**<T\>(`newChild`: T, `refChild`: Node): T

###### Type parameters:

Name | Type |
:------ | :------ |
`T` | Node |

###### Parameters:

Name | Type |
:------ | :------ |
`newChild` | T |
`refChild` | Node |

**Returns:** T

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10851

___

##### isDefaultNamespace

▸ **isDefaultNamespace**(`namespace`: *string*): *boolean*

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10852

___

##### isEqualNode

▸ **isEqualNode**(`otherNode`: Node): *boolean*

Returns whether node and otherNode have the same properties.

###### Parameters:

Name | Type |
:------ | :------ |
`otherNode` | Node |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10856

___

##### isSameNode

▸ **isSameNode**(`otherNode`: Node): *boolean*

###### Parameters:

Name | Type |
:------ | :------ |
`otherNode` | Node |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10857

___

##### lookupNamespaceURI

▸ **lookupNamespaceURI**(`prefix`: *string*): *string*

###### Parameters:

Name | Type |
:------ | :------ |
`prefix` | *string* |

**Returns:** *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10858

___

##### lookupPrefix

▸ **lookupPrefix**(`namespace`: *string*): *string*

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |

**Returns:** *string*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10859

___

##### matches

▸ **matches**(`selectors`: *string*): *boolean*

Returns true if matching selectors against element's root yields element, and false otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | *string* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5185

___

##### msGetRegionContent

▸ **msGetRegionContent**(): *any*

**Returns:** *any*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5186

___

##### normalize

▸ **normalize**(): *void*

Removes empty exclusive Text nodes and concatenates the data of remaining contiguous exclusive Text nodes into the first of their nodes.

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10863

___

##### prepend

▸ **prepend**(...`nodes`: (*string* \| Node)[]): *void*

Inserts nodes before the first child of node, while replacing strings in nodes with equivalent Text nodes.

Throws a "HierarchyRequestError" DOMException if the constraints of the node tree are violated.

###### Parameters:

Name | Type |
:------ | :------ |
`...nodes` | (*string* \| Node)[] |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11362

___

##### querySelector

▸ **querySelector**<K\>(`selectors`: K): HTMLElementTagNameMap[K]

Returns the first element that is a descendant of node that matches selectors.

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *object* \| *a* \| *abbr* \| *address* \| *applet* \| *area* \| *article* \| *aside* \| *audio* \| *b* \| *base* \| *basefont* \| *bdi* \| *bdo* \| *blockquote* \| *body* \| *br* \| *button* \| *canvas* \| *caption* \| *cite* \| *code* \| *col* \| *colgroup* \| *data* \| *datalist* \| *dd* \| *del* \| *details* \| *dfn* \| *dialog* \| *dir* \| *div* \| *dl* \| *dt* \| *em* \| *embed* \| *fieldset* \| *figcaption* \| *figure* \| *font* \| *footer* \| *form* \| *frame* \| *frameset* \| *h1* \| *h2* \| *h3* \| *h4* \| *h5* \| *h6* \| *head* \| *header* \| *hgroup* \| *hr* \| *html* \| *i* \| *iframe* \| *img* \| *input* \| *ins* \| *kbd* \| *label* \| *legend* \| *li* \| *link* \| *main* \| *map* \| *mark* \| *marquee* \| *menu* \| *meta* \| *meter* \| *nav* \| *noscript* \| *ol* \| *optgroup* \| *option* \| *output* \| *p* \| *param* \| *picture* \| *pre* \| *progress* \| *q* \| *rp* \| *rt* \| *ruby* \| *s* \| *samp* \| *script* \| *section* \| *select* \| *slot* \| *small* \| *source* \| *span* \| *strong* \| *style* \| *sub* \| *summary* \| *sup* \| *table* \| *tbody* \| *td* \| *template* \| *textarea* \| *tfoot* \| *th* \| *thead* \| *time* \| *title* \| *tr* \| *track* \| *u* \| *ul* \| *var* \| *video* \| *wbr* \| *emoji-picker* |

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | K |

**Returns:** HTMLElementTagNameMap[K]

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11366

▸ **querySelector**<K\>(`selectors`: K): SVGElementTagNameMap[K]

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *symbol* \| *a* \| *script* \| *style* \| *title* \| *circle* \| *clipPath* \| *defs* \| *desc* \| *ellipse* \| *feBlend* \| *feColorMatrix* \| *feComponentTransfer* \| *feComposite* \| *feConvolveMatrix* \| *feDiffuseLighting* \| *feDisplacementMap* \| *feDistantLight* \| *feFlood* \| *feFuncA* \| *feFuncB* \| *feFuncG* \| *feFuncR* \| *feGaussianBlur* \| *feImage* \| *feMerge* \| *feMergeNode* \| *feMorphology* \| *feOffset* \| *fePointLight* \| *feSpecularLighting* \| *feSpotLight* \| *feTile* \| *feTurbulence* \| *filter* \| *foreignObject* \| *g* \| *image* \| *line* \| *linearGradient* \| *marker* \| *mask* \| *metadata* \| *path* \| *pattern* \| *polygon* \| *polyline* \| *radialGradient* \| *rect* \| *stop* \| *svg* \| *switch* \| *text* \| *textPath* \| *tspan* \| *use* \| *view* |

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | K |

**Returns:** SVGElementTagNameMap[K]

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11367

▸ **querySelector**<E\>(`selectors`: *string*): E

###### Type parameters:

Name | Type | Default |
:------ | :------ | :------ |
`E` | Element | Element |

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | *string* |

**Returns:** E

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11368

___

##### querySelectorAll

▸ **querySelectorAll**<K\>(`selectors`: K): *NodeListOf*<HTMLElementTagNameMap[K]\>

Returns all element descendants of node that match selectors.

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *object* \| *a* \| *abbr* \| *address* \| *applet* \| *area* \| *article* \| *aside* \| *audio* \| *b* \| *base* \| *basefont* \| *bdi* \| *bdo* \| *blockquote* \| *body* \| *br* \| *button* \| *canvas* \| *caption* \| *cite* \| *code* \| *col* \| *colgroup* \| *data* \| *datalist* \| *dd* \| *del* \| *details* \| *dfn* \| *dialog* \| *dir* \| *div* \| *dl* \| *dt* \| *em* \| *embed* \| *fieldset* \| *figcaption* \| *figure* \| *font* \| *footer* \| *form* \| *frame* \| *frameset* \| *h1* \| *h2* \| *h3* \| *h4* \| *h5* \| *h6* \| *head* \| *header* \| *hgroup* \| *hr* \| *html* \| *i* \| *iframe* \| *img* \| *input* \| *ins* \| *kbd* \| *label* \| *legend* \| *li* \| *link* \| *main* \| *map* \| *mark* \| *marquee* \| *menu* \| *meta* \| *meter* \| *nav* \| *noscript* \| *ol* \| *optgroup* \| *option* \| *output* \| *p* \| *param* \| *picture* \| *pre* \| *progress* \| *q* \| *rp* \| *rt* \| *ruby* \| *s* \| *samp* \| *script* \| *section* \| *select* \| *slot* \| *small* \| *source* \| *span* \| *strong* \| *style* \| *sub* \| *summary* \| *sup* \| *table* \| *tbody* \| *td* \| *template* \| *textarea* \| *tfoot* \| *th* \| *thead* \| *time* \| *title* \| *tr* \| *track* \| *u* \| *ul* \| *var* \| *video* \| *wbr* \| *emoji-picker* |

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | K |

**Returns:** *NodeListOf*<HTMLElementTagNameMap[K]\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11372

▸ **querySelectorAll**<K\>(`selectors`: K): *NodeListOf*<SVGElementTagNameMap[K]\>

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *symbol* \| *a* \| *script* \| *style* \| *title* \| *circle* \| *clipPath* \| *defs* \| *desc* \| *ellipse* \| *feBlend* \| *feColorMatrix* \| *feComponentTransfer* \| *feComposite* \| *feConvolveMatrix* \| *feDiffuseLighting* \| *feDisplacementMap* \| *feDistantLight* \| *feFlood* \| *feFuncA* \| *feFuncB* \| *feFuncG* \| *feFuncR* \| *feGaussianBlur* \| *feImage* \| *feMerge* \| *feMergeNode* \| *feMorphology* \| *feOffset* \| *fePointLight* \| *feSpecularLighting* \| *feSpotLight* \| *feTile* \| *feTurbulence* \| *filter* \| *foreignObject* \| *g* \| *image* \| *line* \| *linearGradient* \| *marker* \| *mask* \| *metadata* \| *path* \| *pattern* \| *polygon* \| *polyline* \| *radialGradient* \| *rect* \| *stop* \| *svg* \| *switch* \| *text* \| *textPath* \| *tspan* \| *use* \| *view* |

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | K |

**Returns:** *NodeListOf*<SVGElementTagNameMap[K]\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11373

▸ **querySelectorAll**<E\>(`selectors`: *string*): *NodeListOf*<E\>

###### Type parameters:

Name | Type | Default |
:------ | :------ | :------ |
`E` | Element | Element |

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | *string* |

**Returns:** *NodeListOf*<E\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:11374

___

##### releasePointerCapture

▸ **releasePointerCapture**(`pointerId`: *number*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`pointerId` | *number* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5187

___

##### remove

▸ **remove**(): *void*

Removes node.

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:3508

___

##### removeAttribute

▸ **removeAttribute**(`qualifiedName`: *string*): *void*

Removes element's first attribute whose qualified name is qualifiedName.

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5191

___

##### removeAttributeNS

▸ **removeAttributeNS**(`namespace`: *string*, `localName`: *string*): *void*

Removes element's attribute whose namespace is namespace and local name is localName.

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |
`localName` | *string* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5195

___

##### removeAttributeNode

▸ **removeAttributeNode**(`attr`: Attr): Attr

###### Parameters:

Name | Type |
:------ | :------ |
`attr` | Attr |

**Returns:** Attr

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5196

___

##### removeChild

▸ **removeChild**<T\>(`oldChild`: T): T

###### Type parameters:

Name | Type |
:------ | :------ |
`T` | Node |

###### Parameters:

Name | Type |
:------ | :------ |
`oldChild` | T |

**Returns:** T

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10864

___

##### removeEventListener

▸ **removeEventListener**<K\>(`type`: K, `listener`: (`ev`: *EmojiPickerEventMap*[K]) => *any*, `options?`: *boolean* \| EventListenerOptions): *void*

###### Type parameters:

Name | Type |
:------ | :------ |
`K` | *emoji-click* \| *skin-tone-change* |

###### Parameters:

Name | Type |
:------ | :------ |
`type` | K |
`listener` | (`ev`: *EmojiPickerEventMap*[K]) => *any* |
`options?` | *boolean* \| EventListenerOptions |

**Returns:** *void*

Defined in: src/types/picker.ts:35

___

##### replaceChild

▸ **replaceChild**<T\>(`newChild`: Node, `oldChild`: T): T

###### Type parameters:

Name | Type |
:------ | :------ |
`T` | Node |

###### Parameters:

Name | Type |
:------ | :------ |
`newChild` | Node |
`oldChild` | T |

**Returns:** T

Defined in: node_modules/typescript/lib/lib.dom.d.ts:10865

___

##### replaceWith

▸ **replaceWith**(...`nodes`: (*string* \| Node)[]): *void*

Replaces node with nodes, while replacing strings in nodes with equivalent Text nodes.

Throws a "HierarchyRequestError" DOMException if the constraints of the node tree are violated.

###### Parameters:

Name | Type |
:------ | :------ |
`...nodes` | (*string* \| Node)[] |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:3514

___

##### requestFullscreen

▸ **requestFullscreen**(`options?`: FullscreenOptions): *Promise*<void\>

Displays element fullscreen and resolves promise when done.

When supplied, options's navigationUI member indicates whether showing navigation UI while in fullscreen is preferred or not. If set to "show", navigation simplicity is preferred over screen space, and if set to "hide", more screen space is preferred. User agents are always free to honor user preference over the application's. The default value "auto" indicates no application preference.

###### Parameters:

Name | Type |
:------ | :------ |
`options?` | FullscreenOptions |

**Returns:** *Promise*<void\>

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5202

___

##### requestPointerLock

▸ **requestPointerLock**(): *void*

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5203

___

##### scroll

▸ **scroll**(`options?`: ScrollToOptions): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`options?` | ScrollToOptions |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5204

▸ **scroll**(`x`: *number*, `y`: *number*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`x` | *number* |
`y` | *number* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5205

___

##### scrollBy

▸ **scrollBy**(`options?`: ScrollToOptions): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`options?` | ScrollToOptions |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5206

▸ **scrollBy**(`x`: *number*, `y`: *number*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`x` | *number* |
`y` | *number* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5207

___

##### scrollIntoView

▸ **scrollIntoView**(`arg?`: *boolean* \| ScrollIntoViewOptions): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`arg?` | *boolean* \| ScrollIntoViewOptions |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5208

___

##### scrollTo

▸ **scrollTo**(`options?`: ScrollToOptions): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`options?` | ScrollToOptions |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5209

▸ **scrollTo**(`x`: *number*, `y`: *number*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`x` | *number* |
`y` | *number* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5210

___

##### setAttribute

▸ **setAttribute**(`qualifiedName`: *string*, `value`: *string*): *void*

Sets the value of element's first attribute whose qualified name is qualifiedName to value.

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |
`value` | *string* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5214

___

##### setAttributeNS

▸ **setAttributeNS**(`namespace`: *string*, `qualifiedName`: *string*, `value`: *string*): *void*

Sets the value of element's attribute whose namespace is namespace and local name is localName to value.

###### Parameters:

Name | Type |
:------ | :------ |
`namespace` | *string* |
`qualifiedName` | *string* |
`value` | *string* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5218

___

##### setAttributeNode

▸ **setAttributeNode**(`attr`: Attr): Attr

###### Parameters:

Name | Type |
:------ | :------ |
`attr` | Attr |

**Returns:** Attr

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5219

___

##### setAttributeNodeNS

▸ **setAttributeNodeNS**(`attr`: Attr): Attr

###### Parameters:

Name | Type |
:------ | :------ |
`attr` | Attr |

**Returns:** Attr

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5220

___

##### setPointerCapture

▸ **setPointerCapture**(`pointerId`: *number*): *void*

###### Parameters:

Name | Type |
:------ | :------ |
`pointerId` | *number* |

**Returns:** *void*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5221

___

##### toggleAttribute

▸ **toggleAttribute**(`qualifiedName`: *string*, `force?`: *boolean*): *boolean*

If force is not given, "toggles" qualifiedName, removing it if it is present and adding it if it is not present. If force is true, adds qualifiedName. If force is false, removes qualifiedName.

Returns true if qualifiedName is now present, and false otherwise.

###### Parameters:

Name | Type |
:------ | :------ |
`qualifiedName` | *string* |
`force?` | *boolean* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5227

___

##### webkitMatchesSelector

▸ **webkitMatchesSelector**(`selectors`: *string*): *boolean*

###### Parameters:

Name | Type |
:------ | :------ |
`selectors` | *string* |

**Returns:** *boolean*

Defined in: node_modules/typescript/lib/lib.dom.d.ts:5228


<!-- database API end -->

### Events

#### `emoji-click`

The `emoji-click` event is fired when an emoji is selected by the user. Example format:

```javascript
{
  emoji: {
    annotation: 'thumbs up',
    group: 1,
    order: 280,
    shortcodes: ['thumbsup', '+1', 'yes'],
    tags: ['+1', 'hand', 'thumb', 'up'],
    unicode: '👍️',
    version: 0.6,
    skins: [
      { tone: 1, unicode: '👍🏻', version: 1 },
      { tone: 2, unicode: '👍🏼', version: 1 },
      { tone: 3, unicode: '👍🏽', version: 1 },
      { tone: 4, unicode: '👍🏾', version: 1 },
      { tone: 5, unicode: '👍🏿', version: 1 }
    ]
  },
  skinTone: 4,
  unicode: '👍🏾'
}
```

And usage:

```js
picker.addEventListener('emoji-click', event => {
  console.log(event.detail); // will log something like the above
});
```

Note that `unicode` will represent whatever the emoji should look like
with the given `skinTone`. If the `skinTone` is 0, or if the emoji has
no skin tones, then no skin tone is applied to `unicode`.

#### `skin-tone-change`

This event is fired whenever the user selects a new skin tone. Example format:

```js
{
  skinTone: 5
}
```

And usage:

```js
picker.addEventListener('skin-tone-change', event => {
  console.log(event.detail); // will log something like the above
})
```

Note that skin tones are an integer from 0 (default) to 1 (light) through 5 (dark).

### Custom emoji

Both the Picker and the Database support custom emoji. Unlike regular emoji, custom emoji
are kept in-memory. (It's assumed that they're small, and they might frequently change, so
there's not much point in storing them in IndexedDB.)

Custom emoji should follow the format:

```js
[
  {
    name: 'Garfield',
    shortcodes: ['garfield'],
    url: 'http://example.com/garfield.png',
    category: 'Cats'
  },
  {
    name: 'Heathcliff',
    shortcodes: ['heathcliff'],
    url: 'http://example.com/heathcliff.png',
    category: 'Cats'
  },
  {
    name: 'Scooby-Doo',
    shortcodes: ['scooby'],
    url: 'http://example.com/scooby.png',
    category: 'Dogs'
  }  
]
```

Note that names are assumed to be unique (case-insensitive), and it's assumed that the `shortcodes` have at least one entry.

The `category` is optional. If you don't provide it, then the custom emoji will appear in a
single category called "Custom".

To pass custom emoji into the `Picker`:

```js
const picker = new Picker({
  customEmoji: [ /* ... */ ]
});
```

Or the `Database`:

```js
const database = new Database({
  customEmoji: [ /* ... */ ]
});
```

Custom emoji can also be set at runtime:

```js
picker.customEmoji = [ /* ... */ ];
database.customEmoji = [ /* ... */ ];
```

### Tree-shaking

If you want to import the `Database` without the `Picker`, or you want to code-split them separately, then do:

```js
import Picker from 'emoji-picker-element/picker';
import Database from 'emoji-picker-element/database';
```

The reason for this is that `Picker` automatically registers itself as a custom element, following [web component best practices](https://justinfagnani.com/2019/11/01/how-to-publish-web-components-to-npm/). But this adds side effects, so bundlers like Webpack and Rollup do not tree-shake as well, unless the modules are imported from completely separate files.

### Within a Svelte project

`emoji-picker-element` is explicitly designed as a custom element, and won't work
as a direct Svelte component. However, if you're already using Svelte 3, then you
can avoid importing Svelte twice by using:

```js
import Picker from 'emoji-picker-element/svelte';
```

`svelte.js` is the same as `picker.js`, except it `import`s Svelte rather than bundling it.

## Data and offline

### Data source and JSON format

If you'd like to host the emoji JSON yourself, you can do:

    npm install emoji-picker-element-data@^1

Then host `node_modules/emoji-picker-element-data/en/emojibase/data.json` (or other JSON files) on your web server.

### Shortcodes

There is no standard for shortcodes, so unlike other emoji data, there is some disagreement as to what a "shortcode" actually is.

`emoji-picker-element-data` is based on `emojibase-data`, which offers several shortcode packs per language. For instance,
you may choose shortcodes from GitHub, Slack, Discord, or Emojibase (the default). You
can browse the available data files [on jsdelivr](https://www.jsdelivr.com/package/npm/emoji-picker-element-data) and see
more details on shortcodes [in the Emojibase docs](https://emojibase.dev/docs/shortcodes).

### Cache performance

For optimal cache performance, it's recommended that your server expose an `ETag` header. If so, `emoji-picker-element` can avoid re-downloading the entire JSON file over and over again. Instead, it will do a `HEAD` request and just check the `ETag`.

If the server hosting the JSON file is not the same as the one containing the emoji picker, then the cross-origin server will also need to expose `Access-Control-Allow-Origin: *` and `Access-Control-Allow-Headers: ETag` (or `Access-Control-Allow-Headers: *` ). `jsdelivr` already does this, which is partly why it is the default.

Note that [Safari does not currently support `Access-Control-Allow-Headers: *`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers#Browser_compatibility), but it does support `Access-Control-Allow-Headers: ETag`.

If `emoji-picker-element` cannot use the `ETag` for any reason, it will fall back to the less performant option, doing a full `GET` request on every page load.

### emojibase-data compatibility (deprecated)

_**Deprecated:** in v1.3.0, `emoji-picker-element` switched from [`emojibase-data`](https://github.com/milesj/emojibase) to
[`emoji-picker-element-data`](https://npmjs.com/package/emoji-picker-element-data) as its default data source. You can still use `emojibase-data`, but only v5 is supported, not v6. Support may be removed in a later release._

When using `emojibase-data`, you must use the _full_ [`emojibase-data`](https://github.com/milesj/emojibase) JSON file, not the "compact" one (i.e. `data.json`, not `compact.json`).

### Trimming the emoji data (deprecated)

_**Deprecated:** in v1.3.0, `emoji-picker-element` switched from [`emojibase-data`](https://github.com/milesj/emojibase) to
[`emoji-picker-element-data`](https://npmjs.com/package/emoji-picker-element-data) as its default data source. With the new `emoji-picker-element-data`, there is no need to trim the emoji down to size. This function is deprecated and may be removed eventually._

If you are hosting the `emojibase-data` JSON file yourself and would like it to be as small as possible, then you can use the utility `trimEmojiData` function:

```js
import trimEmojiData from 'emoji-picker-element/trimEmojiData.js';
import emojiBaseData from 'emojibase-data/en/data.json';

const trimmedData = trimEmojiData(emojiBaseData);
```

Or if your version of Node doesn't support ES modules:

```js
const trimEmojiData = require('emoji-picker-element/trimEmojiData.cjs');
```

### Offline-first

`emoji-picker-element` uses a "stale while revalidate" strategy to update emoji data. In other words, it will use any existing data it finds in IndexedDB, and lazily update via the `dataSource` in case that data has changed. This means it will work [offline-first](http://offlinefirst.org/) the second time it runs.

If you would like to manage the database yourself (e.g. to ensure that it's correctly populated before displaying the `Picker`), then create a new `Database` instance and wait for its `ready()` promise to resolve:

```js
const database = new Database();
try {
  await database.ready();
} catch (err) {
  // Deal with any errors (e.g. offline)
}
```

If `emoji-picker-element` fails to fetch the JSON data the first time it loads, then it will display an error message.

### Environments without IndexedDB

`emoji-picker-element` has a hard requirement on [IndexedDB](https://developer.mozilla.org/en-US/docs/Glossary/IndexedDB), and will not work without it.

For browsers that don't support IndexedDB, such as [Firefox in private browsing mode](https://bugzilla.mozilla.org/show_bug.cgi?id=1639542), you can polyfill it using [fake-indexeddb](https://github.com/dumbmatter/fakeIndexedDB). Here is [a working example](https://bl.ocks.org/nolanlawson/651e6fbe4356ff098f505e6cc5fb8cd8) and [more details](https://github.com/nolanlawson/emoji-picker-element/issues/9).

For Node.js environments such as [Jest](https://jestjs.io/) or [JSDom](https://github.com/jsdom/jsdom), you can also use fake-indexeddb. A [working example](https://github.com/nolanlawson/emoji-picker-element/blob/d45b61b559b8cef6840b7036e3d1749a213c490e/config/jest.setup.js#L15-L18) can be found in the tests for this very project.

## Design decisions

Some of the reasoning behind why `emoji-picker-element` is built the way it is.

### IndexedDB

The [`emojibase-data`](https://github.com/milesj/emojibase) English JSON file is [854kB](https://unpkg.com/browse/emojibase-data@5.0.1/en/), and the "compact" version is still 543kB. That's a lot of data to keep in memory just for an emoji picker. And it's not as if that number is ever going down; the Unicode Consortium keeps adding more emoji every year.

Using IndexedDB has a few advantages:

1. We don't need to keep the full emoji data in memory at all times.
2. After the first load, there is no need to download, parse, and index the JSON file again, because it's already available in IndexedDB.
3. If you want, you can even [load the IndexedDB data in a web worker](https://github.com/nolanlawson/emoji-picker-element/blob/ff86a42/test/adhoc/worker.js), keeping the main thread free from non-UI data processing.

### Native emoji

To avoid downloading a large sprite sheet that renders a particular emoji set – which may look out-of-place on different platforms, or may have [IP issues](https://blog.emojipedia.org/apples-emoji-crackdown/) – `emoji-picker-element` only renders native emoji. This means it is limited to the emoji actually installed on the user's device.

To avoid rendering ugly unsupported or half-supported emoji, `emoji-picker-element` will automatically detect emoji support and only render the supported characters. (So no empty boxes or awkward double emoji.) If no color emoji are supported by the browser/OS, then an error message is displayed (e.g. older browsers, some odd Linux configurations).

### JSON loading

Browsers deal with JSON more efficiently when it's loaded via `fetch()` rather than embedded in JavaScript. It's
[faster for the browser to parse JSON than JavaScript](https://joreteg.com/blog/improving-redux-state-transfer-performance),
becuase the data is being parsed in the more tightly-constrained JSON format than the generic JavaScript format.

Plus, embedding the JSON directly would mean re-parsing the entire object on second load, which is something we want to avoid since the data is already in IndexedDB.

### Browser support

`emoji-picker-element` only supports the latest versions of Chrome, Firefox, and Safari, as well as equivalent browsers (Edge, Opera, etc.). If you need support for older browsers, you will need polyfills for the following things (non-exhaustive list):

- Custom elements
- Shadow DOM
- ES2019+

That said, older browsers may not have a color emoji font installed at all, so `emoji-picker-element` will not work in those cases.

## Contributing

See [CONTRIBUTING.md](https://github.com/nolanlawson/emoji-picker-element/blob/master/CONTRIBUTING.md).
