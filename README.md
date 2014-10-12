# Table of Contents (TOC) Generator

Automatically generate table of contents for a given area of content.

## Settings

| Name | Description |
| --- | --- |
| `where` | Reference to the container that will hold the table of contents. |
| `content` | Reference to the content container. |
| `itemFormatter` | See [Item Formatter](#item-formatter). |
| `anchorFormatter` | See [Anchor Name](#anchor-name). |
| `offsetCalculator` | See [Offset Line of Sight](#offset-line-of-sight). |

### Example

```
var contents;

contents = $.gajus
    .contents({
        where: $('#toc'),
        content: $('article')
    });
```

## Events

Use the generated list element to lister and trigger events.

### `change.gajus.contents`

* The page is loaded.
* User navigates to a new section of the page.

The second parameter of the event callback has reference to the current and previous (if any) active heading and anchor.

```js
contents
    .on('change.gajus.contents', function (event, change) {
        if (change.previous) {
            change.previous.heading.removeClass('active-heading');
            change.previous.anchor.removeClass('active-anchor');
        }

        change.current.heading.addClass('active-heading');
        change.current.anchor.addClass('active-anchor');
    });
```

### `resize.gajus.contents`

* The page is loaded.
* In response to "resize" window event.
* In response to "orientationchange" window event.

This event is used to recalculate heading offset variables.

```js
contents
    .on('resize.gajus.contents', function (event) {
        
    });
```

You can manually trigger `resize.gajus.contents` event, e.g. if you have programmatically interfered with the content.

contents
    .trigger('resize.gajus.contents');

## Markup

Table of contents is a list element. The list is nested to represent the heading hierarchy. The default behavior is to represent each heading using a hyperlink, e.g.

```html
<h1>JavaScript</h1>
<h2>History</h2>
<h2>Trademark</h2>
<h2>Features</h2>
<h3>Imperative and structured</h3>
<h3>Dynamic</h3>
<h3>Functional</h3>
<h2>Syntax</h2>
```

The above content will generate the following table of contents:

```html
<ul>
    <li>
        <a href="javascript">JavaScript</a>

        <ul>
            <li>
                <a href="history">History</a>
            </li>
            <li>
                <a href="trademark">Trademark</a>
            </li>
            <li>
                <a href="features">Features</a>

                <ul>
                    <li>
                        <a href="imperative-and-structured">Imperative and structured</a>
                    </li>
                    <li>
                        <a href="dynamic">Dynamic</a>
                    </li>
                    <li>
                        <a href="functional">Functional</a>
                    </li>
                </ul>
            </li>
            <li>
                <a href="syntax">Syntax</a>
            </li>
        </ul>
    </li>
</ul>
```

### Item Formatter

Item formatter is used to represent each item in the list.

The default item formatter implementation:

1. Wraps text of each heading in a hyperlink element.
2. Uses heading "id" attribute to link hyperlink to the heading anchor.
3. Appends the hyperlink to the list.

```js
/**
 * @param {jQuery} li List element.
 * @param {jQuery} heading Heading element.
 */
$.gajus.contents.generateHeadingHierarchyList.itemFormatter = function (li, heading) {
    var hyperlink = $('<a>');

    hyperlink.text(heading.text());
    hyperlink.attr('href', '#' + heading.attr('id'));

    li.append(hyperlink);
};
```

You can overwrite this behavior using a custom `itemFormatter` function:

```js
$.gajus
    .contents({
        itemFormatter: function (li, heading) {}
    });
```

### Anchor Name

The default implementation relies on each heading having an "id" attribute to enable anchor navigation. If heading does not have "id", `$.gajus.contents.anchorFormatter` will be used to derive the value from the heading text.

```js
/**
 * Format text into ID/anchor safe value.
 *
 * @see http://stackoverflow.com/a/1077111/368691
 * @param {String} str Arbitrary string.
 * @return {String}
 */
$.gajus.contents.anchorFormatter = function (str) {
    return str
        .toLowerCase()
        .replace(/[ãàáäâ]/g, 'a')
        .replace(/[ẽèéëê]/g, 'e')
        .replace(/[ìíïî]/g, 'i')
        .replace(/[õòóöô]/g, 'o')
        .replace(/[ùúüû]/g, 'u')
        .replace(/[ñ]/g, 'n')
        .replace(/[ç]/g, 'c')
        .replace(/\s+/g, '-')
        .replace(/[^a-z0-9\-_]+/g, '-')
        .replace(/\-+/g, '-')
        .replace(/^\-|\-$/g, '')
        .replace(/^[^a-z]+/g, '');
};
```

You can overwrite this behavior using a custom `anchorFormatter` function:

```js
$.gajus
    .contents({
        anchorFormatter: function (str) {}
    });
```

#### Solving ID Conflicts

If there are multiple headings with the same name, e.g.

```html
<h2>Allow me to reiterate</h2>
<h2>Allow me to reiterate</h2>
<h2>Allow me to reiterate</h2>
```

The mechanism responsible for ensuring that only unique IDs are assigned will suffix each value with an incremental index:

```html
<h2 id="allow-me-to-reiterate">Allow me to reiterate</h2>
<h2 id="allow-me-to-reiterate-1">Allow me to reiterate</h2>
<h2 id="allow-me-to-reiterate-2">Allow me to reiterate</h2>
```

This operation is performed after `anchorFormatter`.

## Offset Line of Sight

The content is considered "in sight" when the heading is at or above the "line of sight". This line of sight can be either top of the page or an arbitrary offset.

The default behavior is to calculate the line of sight as 1/3 of the window height.

```js
/**
 * @return {Number}
 */
$.gajus.contents.offsetIndex.offsetCalculator = function () {
    return $(window).height() / 3;
};
```

You can overwrite the function used to calculate the offset with the `offsetCalculator` setting:

```js
$.gajus
    .contents({
        offsetCalculator: function () {
            // The heading must be at the most 20px from the top of the screen.
            return 20;
        }
    });
```

The function to calculate the line of sight is called upon initiation and in response to window resize event.