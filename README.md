1. [Introduction](#introduction)
2. [BEM](#bem)
3. [Reusable classes](#reusable-components)
4. [Architecture](#architecture)
5. [Selectors](#selectors)
6. [Theory](#theory)
7. [Formatting](#formatting)
8. [Whitespace](#whitespace)

# Introduction

The purpose of this document is to specify a CSS structure that improves the componentisation and encapsulation of Caplin CSS.

A mix of [BEM](https://en.bem.info/), [OOCSS](http://appendto.com/2014/04/oocss/) and [SuitCSS](http://suitcss.github.io/) has been chosen as the basis.

Using BEM will mean that all CSS in a blade is encapsulated and can be modified or deleted at will.




# BEM

BEM stands for Block Element Modifier. It looks like this:

``` html
<div class="Filter">
	<div class="Filter-row" />
	<div class="Filter-row Filter-row--highlight" />
	<div class="Filter-row" />
</div>
```

A Block is a reusable piece of intertwined HTML and CSS. 
In this example, we are creating a *Filter*.

An Element is a child DOM element that forms an essential part of the component we are trying to build.

So `Filter-row` is an Element which will uniquely style rows inside `Filter` Blocks.

A Modifier is a variant of either of a Block or one of the Elements it contains.
In this example, we want the first row to stand out, hence the modifier of `--highlight`.


*The syntax defined here differs from the original BEM (see [SuitCSS](https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md)), but the end result is the same.*


### Blocks
The single root element for every Block.

The Block is be named with PascalCase (camelCase with a capitalised first letter).
Every selector related to the Block is prepended by the Block name, this namespaces the Block and ensures styles do not leak or conflict with other parts of the page.
``` html
<div class="Filter" />
```
``` css
.Filter {...}
```


### Elements
Elements within a Block are named with camelCase and separated from the Block by a hyphen.
``` html
<div class="Filter">
	<div class="Filter-element" />
	<div class="Filter-element" />
	<div class="Filter-anotherElement" />
</div>
```
``` css
.Filter-element {...}
.Filter-anotherElement {...}
```

**Note: The BEM 'tree' does not replicate the DOM tree. Elements within Elements must *not* be named as such.**

``` html
<!-- Wrong -->
<div class="Filter">
	<div class="Filter-outerElement">
		<div class="Filter-outerElement-innerElement" />
	</div>
</div>
```

``` css
/* Wrong */
.Filter-outerElement { ... }
.Filter-outerElement-innerElement { ... }
```

```html
<!-- Correct -->
<div class="Filter">
	<div class="Filter-outerElement">
		<div class="Filter-innerElement" />
	</div>
</div>
```
``` css
/* Correct */
.Filter-outerElement { ... }
.Filter-innerElement { ... }
```

**Pseudo classes can be used ([assuming they are supported](http://www.quirksmode.org/css/selectors/)) to select elements if it is semantically correct to do so.**

``` css
.Filter-element { margin-top: 10px; }
.Filter-element:first-child { margin-top: 0; }
```

### Modifiers
Modifiers (variants) follow the same naming as Elements, but are separated by a double hyphen.
``` html
<div class="Filter--large" />
```
``` css
.Filter {...}
.Filter--large { height: 9001px; }
```

You can have modifiers on Elements:

``` html
<div class="Filter">
	<div class="Filter-element" />
	<div class="Filter-element Filter-element--large" />
	<div class="Filter-element" />
</div>
```
``` css
.Filter-element {...}
.Filter-element--large { height: 9001px; }
```

### States
``` html
<div class="Filter is-selected" />
```
``` css
.Filter { ... }
.Filter.is-selected { border: red; }
```
**The State class must be chained with the relevant Block or Element.**

If this is not done, encapsulation will be broken.

``` css
/* Forbidden. */
.Filter { ... }
.is-selected { ... }
```

States are a variation on modifiers - in the original BEM, there is no differentiation.

If the class is added or removed over the life of the element (via JavaScript), it is a State.


**States must not be included in the HTML or template markup.**

Assuming a popup should be hidden on page load:

**Good:**
``` css
.Popout { display: none; }
.Popout.is-visible { display: block; }
```

``` html
<!-- HTML/Template file -->
<div class="Popout" />
```

**Bad:**
``` css
.Popout { display: block; }
.Popout.is-hidden { display: none; }
```

``` html
<!-- HTML/Template file -->
<div class="Popout is-hidden" />
```


### Nested Blocks

*Blocks can be used inside other Blocks.*

``` html
<!-- Outer Block -->
<div class="Filter">
	<div class="Filter-element">
		 <!-- Inner Block -->
		 <a class="Button" />
	</div>
</div>
```

**Blocks must not depend on their location in the DOM.**

If a Block is dependent on being contained within another Block or element, it can no longer be reused elsewhere in the Application.

In this example, the `Button` Block will not know what styles or dimensions are applied to `.Filter`. Therefore the CSS for a Block must be written in a way that works regardless of the properties of it's container.

The most common unknown will be the dimensions of the parent/container.

**Blocks must not affect child elements.**

``` css
/* Good */
.Filter--xyzElement {display: inline-block; }

/* Bad */
.Filter div {display: inline-block; }
```

As any Block could be placed within any other Block, Blocks must not use selectors that could affect these 'child' Blocks or elements. Use Element classes instead (e.g. `Filter-element`).

**The exception to this rule is when Blocks are explicitly designed to contain other Blocks.** 
*For example, a `ButtonGroup` Block which aligns a group of child `Button` Blocks. See [Selectors#Direct Child](#direct-child).*


# Reusable Components

## Utility Classes

Utility classes must have a single responsibility. They are expected to be used frequently and as such, should rarely be modified after creation. 

``` css
.u-strip {
	display: block !important;
	width: 100% !important;
}
```

*(This is **proactive** use of `!important`, as we know that any element with the class of `strip` should span the entire width of it's parent.)*

``` css
.u-positionCenter {
	top: 0 !important;
	right: 0 !important;
	bottom: 0 !important;
	left: 0 !important;
	margin: auto !important;
	position: absolute !important;
}
```

#### Naming

camelCase prefixed by `u-`, for example:

``` css
.u-myUtilClass {}
```

#### Use
These classes can be added straight onto any element or Block.

``` html
<body>
	<div class="Popup u-positionCenter is-visible">
	</div>
</body>
```

``` css
.Popup {
	display: none;
	height: 100%;
	width: 100%;
	max-height: 400px;
	max-width: 600px;
	z-index: 9001;
}
.Popup.is-visible { display: block }
```


## Skin Classes

Skin classes are similar to Utility classes, but deal with visual styles.

If there are multiple elements or Blocks using similar rules, then we can factor them out.

``` css
.Header {
	...
	background: grey;
	border: 1px solid #BBB;
	border-radius: 2px;
	...
}


.Filter {
	...
	background: grey;
	border: 1px solid #BBB;
	border-radius: 2px;
	...
}
```

Elements can share several properties, however that does not mean these properties should all be extracted to the same skin class. The skin classes must be short (1-4 rules) and only contain related properties.

``` css
.skin-box {
	background: grey;
	border: 1px solid #BBB;
	border-radius: 2px;
}
```

Assuming the UI in our app calls for many elements to have a grey background with a rounded border, this can be reused in different Blocks.

``` html
<div class="Filter skin-box">
	<div class="Filter-element" />
</div>
```

The properties below are *not* closely related and therefore this class cannot be reused in many situations.
``` css
/* Bad */
.skin-text {
	color: black;
	background: white;
	font-family: Helvetica;
	border-radius: 5px;
}
```

Note: It is *not* correct to apply this class and then override some of the properties.

``` html
<div class="Filter skin-main" />
```

``` css
/* Wrong */
.Filter {
	color: red;
	background: white;
}
```

**Using skins classes enforces consistency in the UI and allows the app to be *partially* re-themed.**


#### Naming
camelCase prefixed by `skin-`, for example:

``` css
.skin-border {}
.skin-borderRounded {}
```


## Blocks
For reusable components (e.g. buttons) that require structure, style and (possibly) multiple elements, the BEM/Block conventions must be followed.

As the HTML/CSS will likely be maintained and reused, it must be commented using the [Knyle Style Sheets (KSS)](http://warpspire.com/kss/syntax/) syntax, so a living styleguide can be created for developers to view, edit and copy examples from. A good example of KSS in action is the [Github Styleguide](https://github.com/styleguide/css).

``` css
/*
 Buttons

 The majority of buttons in the site are built from this class.

 Markup: 
 <button class="Button {{modifier_class}}">Button Element</button>

 .Button--small         - Make the button smaller.
 .Button--large         - Make the button larger.
 .Button--default       - Apply default styling to the button.
 .Button--primary		- Apply primary styling to the button.

 Styleguide 1.1
*/

.Button {
	display: inline-block;
	padding: 6px 12px;
	margin-bottom: 0;
	font-size: 14px;
}

.Button--small {
	padding: 5px 10px;
	font-size: 12px;
}

.Button--large {
	padding: 10px 16px;
	font-size: 18px;
}

.Button--default {
	color: #333;
	background-color: #fff;
	border: 1px solid #ccc;
	border-radius: 4px;
}

.Button--primary {
	color: #fff;
	background-color: #337ab7;
	border: 1px solid #2e6da4;
	border-radius: 4px;
}
```


**Any CSS that is context specific must be in a Modifier or State.** Think about reuse in different parts of the App.




# Architecture

Each BEM Block must be in it's own CSS file.

All CSS at the Blade level must be BEM. The encapsulation provided by BEM means that no other elements can be affected by, or become dependent on, the CSS written in a particular blade. You need to be able to add, modify or delete BEM Blocks without being concerned of any impact on other Blocks or parts of the App.

Utility, skin and component classes will be in the `default-aspect` or in external libraries.

If in doubt, write CSS in the Blade with BEM, instead of creating App/Aspect level CSS. App level CSS should rarely be modified once created, whereas it is possible to refactor your BEM CSS.




# Selectors

### Nesting
#### **Don't nest your CSS.**

[*The descendant selector (whitespace).*](https://developer.mozilla.org/en-US/docs/Web/CSS/Descendant_selectors)

``` css
.btn { ... /* Default styling */ }
.Block .btn { ... }
```

``` html
<header class="Block">
	<button class="btn" />
</header>
```

While nesting decreases the classes required in the HTML, the CSS is now dependent on the HTML structure. 

It is often the case that variants of elements are created for a specific situation and later reused elsewhere. In this example, `.btn` cannot be reused outside of `.block`.

Could this be rewritten to make it reusable?

``` css
.btn { ... }
.btn--variantA { ... }
```

``` html
<header class="Block">
	<button class="btn btn--variantA" />
</header>
```




### Direct Child
The direct child selector `>` can be used to improve performance and limit the cascade of styles (good). But what if the DOM needs to be changed and another layer of HTML introduced? Now the CSS needs to be modified for what should be a trivial change.

For example, moving from this:

``` html
<header class="Header">
	<img src="..." />
</header>
```

``` css
.Header > img { ... }
```

To this:
``` html
<header class="Header">
	<div class="grid">
		<img src="..." />
		<p>Lorem ipsum</p>
	</div>
</header>
```

Will break your CSS.

**Use the direct child selector where it makes semantic sense to do so.** 
For example, creating a new BEM Block to contain `.Buttons`:

``` css
.ButtonGroup { ... }

.ButtonGroup > .Button {
	float: left;
	margin-left: 10px;
}

.ButtonGroup > .Button:first-child {
	margin-left: 0;
}
```
``` html
<div class="ButtonGroup">
	<button class="Button"></div>
	<button class="Button"></div>
	<button class="Button"></div>
</div>
```

See: [Bootstrap button groups](http://getbootstrap.com/components/#btn-groups)


### No ID's.
ID's are allowed for JavaScript and fragment identifiers, but should never be used in CSS.

* ID's override any styles applied by classes and then require the use of even more specific selectors to override the ID's rule (and so on).
* Only one ID can exist on a page, therefore the ruleset cannot be used more than once.
* JavaScript ID's cannot be changed or removed for fear of altering styling.

``` css
/* The performance of these are virtually identical. */
#Block { ... }
.Block { ... }
```

### No qualifiers
``` css
/* No. */
ul.Nav { ... }
```

The `ul` qualifier makes this more specific than any other class applied to the element and can only be overridden by an ID (bad) or using !important (bad).

Furthermore, the `.Nav` class cannot be reused on a non-`ul` element, such as a `nav` element.


### No nested wildcards
``` css
.Block * { ... }
```

While the wildcard does not *always* have the significant performance impact it used to, there are very few cases where it is the correct course of action. 

[Non-nested wildcards can be used sparingly.](http://www.paulirish.com/2012/box-sizing-border-box-ftw)


### No reactive !important

!important is the nuclear option when it comes to specificity. If following the rules above, it should never need to be used reactively. That is, to override rule already being applied.

It does have a place when attempting to ensure a class will always work as expected. For example:
``` css
.hidden { display: none !important; }
```

### No null rules

Setting a CSS property to a default or null value is rarely the correct course of action. It is generally a sign that the property has been set too early in the CSS.

``` css
/* Bad */
selector {
	float: none;
	border: none;
	padding: 0;
	margin: 0;
}
```




# Theory

## Selector Intent
The selectors used when creating a rule should match the *intent* behind the rule.

For example, when styling a new navigation (which happens to be in the footer):
``` css
/* This will style every anchor in the footer, when the intent was only to style links in a secondary navigation. */
footer a { ... }

/* Better. The secondary nav can now be used elsewhere on the page. */
.SecondaryNav a { ... }

/* Best. This does not select all anchors in the SecondaryNav, some of which may be used for other purposes. */
.SecondaryNav-link { ... }
```

``` html
<nav class="SecondaryNav">
	<a class="SecondaryNav-link" />
	<a class="SecondaryNav-link" />
	<a class="SecondaryNav-socialIcon" />
</nav>
```

> There are only two hard things in Computer Science: cache invalidation and naming things.

Classes should not describe the exact style being applied, as they cannot be changed:
``` css
.blue { color: blue; }

/* After theming */
.blue { color: red; }
```

Classes should not be contextual, as they cannot be reused:
``` css
.footer-color { color: blue; }
```

``` html
<header>
	<nav>
		<a class="footer-color" />
	</nav>
</header>
```

Aim for a compromise between describing the style and describing the use.

``` css
.Nav-link { color: blue; }
```


## Specificity
Specificity is increased when the descendant selector is used.
``` css
/* This: */
.Block a { ... }

/* Is more specific than this: */
.Block-link { ... }

/*
Now when `.Block-link` needs to override rules from `.Block a`, there will be a specificity war, leading to either: 
*/
header .Block-link { ... }
a.Block-link { ... }
.Block-link { ... !important }

/* And the cycle will continue... */
```

Additionally:

``` css
/* Can no longer be used outside header */
header .Block-link

/* Can no longer be used for non-anchors */
a.Block-link { ... }

/* Will cause specificity problems if reused */
.Block-link { ... !important } 
```


## More BEM
A child element of a Block is only a BEM Element (of that Block), if the element *must* only ever be used within that block.

In our Filter example, we have two buttons inside one of our `Filter-rows`.

``` html
<div class="Filter">
	<div class="Filter-row">
		<button />
		<button />
	</div>
</div>
```

If we wanted to style the buttons, the BEM method would be to add a new class:

``` html
<div class="Filter">
	<div class="Filter-row">
		<button class="Filter-button" />
		<button class="Filter-button" />
	</div>
</div>
```

However this should only be done if the `Filter-buttons` will *only* be used inside a `Filter` Block.

If the buttons are generic and will be used in multiple places, a generic Block (e.g. `.Button`) should be created (if necessary) and used:

``` html
<div class="Filter">
	<div class="Filter-row">
		<button class="Button" />
		<button class="Button" />
	</div>
</div>
```

The `.Button` CSS must be flexible enough to allow the `Button` class to be used in a variety of situations. Aka Location agnostic - not dependent on it's container.

In the majority of cases, elements in a BEM Block will need some generic rulesets *and* some rules specific to their location. Rather than duplicating rules in CSS, the element can be 'composed' with several CSS classes. 

``` html
<div class="Filter">
	<div class="Filter-row">
		<button class="Button Filter-button" />
		<button class="Button Filter-button" />
	</div>
</div>
```

``` css
.Button { ... }

.Filter { ... }
.Filter-row { ... }
.Filter-button.Button { ... }
```

Factoring out the reusable component from the CSS (in this case `.Button`) and using `Filter-row` to apply button styling specifically for a `Filter` Block, removes duplication and let's us reuse the `Button` class. 

TODO: would a normal BEM modifier break encapsulation for this?



# Formatting
For clarity and consistency:
* Indent with tabs. [So it is written, so let it be done](http://caplin.github.io/StyleGuide/).
* Opening brace on selector line.
* Closing brace on new line.
* A single space before the opening bracket.
* A single space after the colon.

``` css
.class {
	property: value;
}
```

These rules simplify diffs by ensuring only one change per line:
* All rules on separate lines.
* All selectors on separate lines.

``` css
.selector1,
.selector2 {
	property: value;
	property: value;
}
```

The only exception is a class with one rule and one selector which performs a specific function. This can be on a single line:

``` css
.red { color: red; }
```

Comments are preceded by a newline, unless at the beginning of a ruleset.
``` css
.class {
	/* Good */
	property: value;
			
	/* Good */
	property: value;
	/* Bad */
	property: value;
}
```

**Wrong:**

``` css
.class
{
	...
}
```
``` css
.class{
	...
}
```
``` css
.class {
	property:value;
}
```



# Whitespace

CSS is minified and stripped of whitespace and comments for production, so make *liberal* use of them.

Group related rules together and separate with newlines:

``` css
.selector {
	/* Box model */
	display: inline-block;
	margin: 0 auto;
	padding: 10px 20px;

	/* Styling */
	color: #FFF;
	background-color: #000;

	/* Fonts */
	font-size: 14px;
	font-weight: bold;
}
```

Separate closely related rulesets by a single line of whitespace:
``` css
.MyButton {
	...
}

.MyButton--large {

}
```

Separate loosly related rulesets by two lines of whitespace:
``` css
.MyButton { ... }

.MyButton--large { ... }


.MyOtherButton { ... }

.MyOtherButton--large { ... }
```

If rulesets are unrelated, they should be in a separate file.

# Further Reading

* [CSS Guidelines](http://cssguidelin.es/)
* [CSS Wizardry - BEM](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/)
* [Smashing Magazine - OOCSS Introduction](http://www.smashingmagazine.com/2011/12/12/an-introduction-to-object-oriented-css-oocss/)
* [CSS Wizardry - Code Smells](http://csswizardry.com/2012/11/code-smells-in-css/)
* [CSS Wizardry - Open/Closed](http://csswizardry.com/2012/06/the-open-closed-principle-applied-to-css/)
* [Drew Barontini - Single Responsibility](http://drewbarontini.com/articles/single-responsibility/)
* [The SASS Way - The Inception Rule](http://thesassway.com/beginner/the-inception-rule)
* [SMACSS - Applicability](https://smacss.com/book/applicability)