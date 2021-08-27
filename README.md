# A Vue component to easily render tabs

[![Latest Version on NPM](https://img.shields.io/npm/v/@jbtje/vue3tabs.svg?style=flat-square)](https://npmjs.com/package/@jbtje/vue3tabs)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![npm](https://img.shields.io/npm/dt/@jbtje/vue3tabs.svg?style=flat-square)](https://www.npmjs.com/package/@jbtje/vue3tabs)

The package contains a [Vue 3](https://vuejs.org/) component to easily display some tabs.

This is how they can be used:
=

```html
<div>
    <tabs :options="{ useUrlFragment: false, defaultTabHash: 'second-tab' }"
          :cache-lifetime="5"

          nav-class="tabs-component-tabs"
          nav-item-active-class="is-active"
          nav-item-class="tabs-component-tab"
          nav-item-disabled-class="is-disabled"
          nav-item-link-active-class="is-active"
          nav-item-link-class="tabs-component-tab-a"
          nav-item-link-disabled-class="is-disabled"
          panels-wrapper-class="tabs-component-panels"
          wrapper-class="tabs-component"

          @changed="tabChanged"
          @clicked="tabClicked"
    >
        <tab id="First-tab"
             name="First-tab"
        >
            This is the content of the first tab
        </tab>
        <tab id="Second-tab"
             name="First-tab"
        >
            This is the content of the second tab
        </tab>
        <tab name="Disabled tab"
             :is-disabled="true"
        >
            This content will be unavailable while :is-disabled prop set to true
        </tab>
        <tab id="this-text-differs"
             name="Custom fragment"
        >
            The fragment that is appended to the url can be customized
        </tab>
        <tab name="Scroll window"
             :link="true"
        >
            When you press this tab, the screen scrolls towards the content of the tab.
        </tab>
        <tab name="Hidden tab"
             :is-hidden="true"
        >
            This tab is not visible, but can be if you set is-hidden to false.
        </tab>
        <tab panel-class="tabs-component-panel"
             prefix="<svg height='20' width='20' viewBox='0 0 128 128' xmlns='http://www.w3.org/2000/svg'><g><path d='m57.362 26.54-37.262 64.535a7.666 7.666 0 0 0 6.639 11.5h74.518a7.666 7.666 0 0 0 6.639-11.5l-37.258-64.535a7.665 7.665 0 0 0 -13.276 0z' fill='#ffb400'/><g fill='#fcf4d9'><rect height='29.377' rx='4.333' width='9.638' x='59.181' y='46.444'/><circle cx='64' cy='87.428' r='4.819'/></g></g></svg>&nbsp;"
             name="Prefix and suffix"
             suffix="&nbsp;<span class='badge'>4</span>"
        >
            A prefix and a suffix can be added
        </tab>
    </tabs>
</div>
```

![Example](https://github.com/JBtje/vue3tabs/raw/master/example.png "Example")

When reloading the page the component will
automatically [display the tab that was previously opened](https://github.com/JBtje/vue3tabs#remembering-the-last-opened-tab).

The rendered output adheres to [the ARIA specification](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Tab_Role).

## Installation

You can install the package via yarn:

```bash
yarn add @jbtje/vue3tabs
```

or npm:

```bash
npm install @jbtje/vue3tabs --save
```

## Usage

The most common use case is to register the components globally:

```js
import {createApp} from 'vue'
import {Tabs, Tab} from '@jbtje/vue3tabs';

createApp( App )
    .component( 'tabs', Tabs )
    .component( 'tab', Tab )
    .mount( '#app' )
```

Alternatively you can do this to register the components:

```js
import Vue         from 'vue';
import {Tabs, Tab} from '@jbtje/vue3tabs';

Vue.component( 'tabs', Tabs );
Vue.component( 'tab', Tab );
```

On your page you can now use html like this to render tabs:

```html
<div>
    <tabs>
        <tab name="First tab">
            First tab content
        </tab>
        <tab name="Second tab">
            Second tab content
        </tab>
        <tab name="Third tab">
            Third tab content
        </tab>
    </tabs>
</div>
```

By default, it will show the first tab.

If you click on a tab a `href` representation of the name will be append to the url. For example clicking on the
tab `Second tab` will append `#second-tab` to the url.

When loading a page with a fragment that matches the `href` of a tab, it will open up that tab. For example
visiting `/#third-tab` will open up the tab with name `Third tab`.

### Remembering the last opened tab

By default, the component will remember which was the last open tab for 5 minutes. If you, for instance, click
on `Third tab` and then visit `/` the third tab will be opened.

You can change the cache lifetime by passing the lifetime in minutes in the `cache-lifetime` property of the `tabs`
component.

```html
<tabs :cache-lifetime="10">
    ...
</tabs>
```

### Auto scrolling the container content

When you press a different tab, the container `scrollWindow` will be scrolled to the top. This only works if you add
overflow to `tabs-component-panels`.

```css
.tabs-component-panels {
   ...
   overflow: auto;
}
```

### Anchor Link

If you set `link` to `true`, the id of the tab will be the same as the hash in the url. Thus creating an Anchor link.
This causes your browser to automatically line up with the top of the content of the tab.
By default, `link` is set to `false`, to prevent the screen from jumping on pressing a tab. The tab id is prefixed with `tab-`.

### Disable modifying the url fragment

When using with other libraries that use the url fragment, you can disable modifying the url fragment by passing
the `useUrlFragment` options. This helps using it with vue-router, or using vue3-tabs-component twice in the same page.

```html

<tabs :options="{ useUrlFragment: false }">
    ...
</tabs>
```

### Callbacks

Tabs have two events to which you can bind: `changed` and `clicked`

```html

<tabs @clicked="tabClicked"
      @changed="tabChanged"
>
    ...
</tabs>
```

For example:

```js
export default {
    methods: {
        tabClicked( selectedTab ) {
            console.log( 'Current tab re-clicked:' + selectedTab.tab.name )
        },
        tabChanged( selectedTab ) {
            console.log( 'Tab changed to:' + selectedTab.tab.name )
        }
    }
}
```

`changed` is emitted when the tab changes and can be used as handle to load data on request.
`clicked` is emitted when an active tab is re-clicked and can be used to e.g. reload the data in the current tab.

### Adding a suffix and a prefix to the tab name

You can add a suffix and a prefix to the tab by using the `suffix` and `prefix` attributes, which can contain HTML.

```html

<tab prefix="my prefix&nbsp;-&nbsp;"
     name="First tab"
     suffix="&nbsp;-&nbsp;my suffix"
>
    First tab content
</tab>
```

The title of the tab will now be `my prefix - First tab - my suffix`.

The fragment that's added to the url when clicking the tab will only be based on the `name` of a tab, the `name-prefix`
and `name-suffix` attributes will be ignored.

### Customizing fragments

When clicking on a tab it's name will be used as a fragment in the url. For example clicking on the `Second tab` will
append `#second-tab` to the current url.

You can customize that fragment by using the `id` attribute.

```html

<div>
    <tabs>
        <tab id="custom-fragment"
             name="My tab"
        >
            First tab content
        </tab>
    </tabs>
</div>
```

Clicking on `My tab` will then append `#custom-fragment` to the url.
Note:

### Setting a default tab

When disabling the cache, it can be useful to specify a default tab to load which is not the first one. You can select
this by passing the `defaultTabHash` option.

```html

<tabs :options="{ defaultTabHash: 'second-tab' }">
    <tab id="first-tab"
         name="First tab"
    >
        First tab content
    </tab>
    <tab id="second-tab"
         name="Default tab"
    >
        Second tab content
    </tab>
</tabs>
```

### CSS

Each node can be styled by specifying classes.

The output HTML classes can be overridden by using the following `Tabs` component attributes:

- `wrapper-class`
- `panels-wrapper-class`
- `nav-class`
- `nav-item-class`
- `nav-item-active-class`
- `nav-item-disabled-class`
- `nav-item-link-class`
- `nav-item-link-active-class`
- `nav-item-link-disabled-class`

The `Tab` content (section) class can be overridden with the `panel-class` attribute

If no custom classes are set, the following classes are used as default:

```html

<div class="tabs-component"
     id="tabs-panels"
> // wrapper-class
    <ul class="tabs-component-tabs"> // nav-class
        <li class="tabs-component-tab is-disabled"> // nav-item-class + nav-item-disabled-class
            <a class="tabs-component-tab-a is-disabled">…</a> // nav-item-link-class + nav-item-link-disabled-class
        </li>
        <li class="tabs-component-tab is-active"> // nav-item-class + nav-item-active-class
            <a class="tabs-component-tab-a is-active">…</a> // nav-item-link-class + nav-item-link-active-class
        </li>
    </ul>
    <div class="tabs-component-panels"
         ref="scrollWindow"
    > // panels-wrapper-class
        <section class="tabs-component-panel"> // Tab > panel-class
            …
        </section>
    </div>
</div>
```

### Example: [style.css](https://github.com/JBtje/vue3tabs/raw/master/style.css)


## Credits

- [Spatie](https://spatie.be)
- [Freek Van der Herten](https://github.com/freekmurze)
- [Willem Van Bockstal](https://github.com/willemvb)
- [Sebastian De Deyne](https://github.com/sebastiandedeyne)
- [Jakub Potocký](https://github.com/Jacobs63/vue3-tabs-component)

**This package is a modified fork of the popular `spatie/vue-tabs-component` Vue 2 package, which has been discontinued by
Spatie**

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.