---
title: Customising Components
description: Todo
---

## Background

The Origami team have two broad aims within the FT:

- Reduce time spent repeating work.
- Unify design across the FT.

One of the ways we meet these aims is by maintaining our shared [component system](https://registry.origami.ft.com/components), along with supporting tools and services.

An Origami component typically consists of SCSS, JavaScript, and demo HTML to be copied into an end project. This approach allows our users to use Origami components within their project regardless of their preferred framework. We maintain and build new components proactively as a team, especially those components which are used across multiple teams and products. But any team may publish an Origami component provided it conforms to the [Origami component specification](https://origami.ft.com/docs/component-spec/) -- and we're here to help.

@todo - Screenshot of ft.com with components highlighted.

![FT.com with components: o-header, o-ads, o-buttons, o-cookie-message](/assets/images/ft.png)
![The Banker with components: o-header, o-ads, o-buttons, o-cookie-message](/assets/images/the-banker.png)
![FT Chinese with components: o-ads, and a custom o-header component](/assets/images/ft-chinese.png)
![Internal documentation with: o-buttons, o-header-services, o-layout](/assets/images/internal-docs.png)

Many products of The Financial Times Group use Origami components to some degree. For example they are used widely across the multiple applications which come together to make [ft.com](https://www.ft.com/). Origami components are also used within our IOS and Android apps, internal products and tools, [FT Chinese](http://www.ftchinese.com/), and other brands under The Financial Times Group such as [The Banker](https://www.thebanker.com/).

@todo - Screenshot of component on multiple platforms.


## Challenge

To accomodate different users, Origami components were themeable. This meant that they contained minimal visual style and expected style to be applied to them from outside. Components could also be theming, meaning that they contained styles designed to change the appearance of another component.

@todo - diagram to demonstrate a theming and themeable component

Unfortunately this theming approach was not widely adopted. And when it was, [ft.com](https://www.ft.com/) specific styles were often added to themeable components rather than to a theme, which would need to be overriden by non-ft.com products.

@todo - Screenshot of striped master brand table. Caption: Our `o-table` component was lagely themeable but included a stripes modifier which coloured alternating rows "pink" and "wheat" for just [ft.com](https://www.ft.com/).

But CSS overrvides increase the bundle size for a project:

<pre><code class="o-syntax-highlight--scss">// Component CSS
.o-table--striped tbody tr:nth-child(even) {
    background-color: pink;
}

// Project CSS
// Oh no! These overrides increase our CSS bundle size.
.o-table--striped tbody tr:nth-child(even) {
    background-color: white;
}
</code></pre>

New [ft.com](https://www.ft.com/) specifc style might be added to a component without notice, visually breaking a non-ft.com site:

 <pre><code class="o-syntax-highlight--scss">// Component CSS
.o-table--striped tbody tr:nth-child(even) {
    background-color: pink;
    border: 1px solid pink;
}

// Project CSS
.o-table--striped tbody tr:nth-child(even) {
    background-color: white;
    // Oh no! We didn't override the border property.
    // And now we have unwanted pink borders in our project.
}
</code></pre>

A CSS selector change in a component could remove overrides or themes without notice:
<pre><code class="o-syntax-highlight--scss">// Component CSS
.o-table.o-table--striped tbody tr:nth-child(even) {
    background-color: pink;
}

// Project CSS
.o-table--striped tbody tr:nth-child(even) {
    // Oh no! An update to the component CSS has added `.o-table` to its selector.
    // Now our project's CSS has less specificity than the component CSS, and doesn't override.
    background-color: white;
}
</code></pre>

And a final issue, when Origami components are included within other components (e.g. a button within a table) they output unqiue component-namespaced class names, which means a component may need to be overriden with the same style multiple times:
<pre><code class="o-syntax-highlight--scss">// Button Component CSS
.o-button {
    background-color: teal;
}
// Table Component CSS
.o-table-button {
    background-color: teal;
}

// Project CSS
// Oh! We have to specify the background colour of our button multiple times.
.o-table-button {
    background-color: white;
}
.o-button {
    background-color: white;
}
</code></pre>

We aimed to find a new approach that would reduce complexity for Origami users, make it easier to build projects of consistent design, improve reliability for non-ft.com projects, and reduce the CSS bundle sizes for all our users.

## Solution

To replace the previous method of "theming", we introduced a new approach we call "branding". Instead of customising a component with external CSS overrides, our new "branding" approach uses a SCSS mixin to configure the component.

And to further our aim to help unify design across the FT, and reduce time spent repeating work, we identified three brands which we would maintain ourselves within our components:

- **master**: FT branding for public ft.com sites and affiliates.
- **internal**: Style suitable for internal products, tools, and documentation.
- **whitelabel**: Base, structural styles only.

@todo - diagram to demonstrate branding in one repo

One way users can include "branded" CSS in their project is using the [Origami Build Service](https://www.ft.com/__origami/service/build/v2/). The build service compiles and minifies CSS for requested components into one bundle. It now supports a `brand` query parameter to return CSS for a specifc brand:

<pre><code class="o-syntax-highlight--html">&lt;!-- o-table CSS for the interal brand -->
&lt;link rel="stylesheet" href="https://www.ft.com/__origami/service/build/v2/bundles/css?modules=o-table@^7.0.6&brand=internal" />
</code></pre>

Alternatively users use a [manual build process](https://origami.ft.com/docs/developer-guide/modules/building-modules/) to compile component SCSS. In this case a `$o-brand` variable is used to select the project's brand:

<pre><code class="o-syntax-highlight--scss">$o-brand: 'internal';
@import 'o-table/main';

@include oTable();
</code></pre>

@todo - demo of branded component

Keeping with SCSS tooling means our components remain accessibile to all our current users, regardless of their choice of framework. We considered using [custom properties (CSS variables)](https://developer.mozilla.org/en-US/docs/Web/CSS/--*) but decided against it for now. We found to provide a fallback for Internet Explorer would bloat our SCSS and compiled CSS; custom properties would also limit our brand configuration to CSS properties only, as apposed to values for use within our SCSS (such as a typography scale to represent a line height and font size, or a boolean on/off to add styles for some brands only).

We've seen how brands help FT developers build projects for the "master", "internal", and "whitelabel" brands. This helps the majority of Origami users, but sites under The Financial Times Group such as [The Banker](https://www.thebanker.com/) will need to create a custom brand. Now we'll see how to create a custom brand.

<pre><code class="o-syntax-highlight--scss">$o-brand: 'mybrand';
@import 'o-table/main';

// Configure the table component for the custom brand "mybrand".
@include oTableDefine($brand: 'mybrand', $config: (
    // Assign custom variable values.
    'variables': (
        table-background: lightcyan,
        table-alternate-background: darkcyan,
        table-border-color: teal,
        table-data-color: darkslategray,
        table-footnote-color: darkslategray
    ),
    // Indicate what features are supported.
    // Only CSS needed for these features are output.
    'supports-variants': (
        'stripes'
    )
));

// Output all supported table CSS.
@include oTable();
</code></pre>

@todo - screenshot of a customised table

