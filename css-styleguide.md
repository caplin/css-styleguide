# CSS Styleguide
1. [Formatting](#formatting)
2. [Whitespace](#whitespace)
3. [Commenting](#commenting)
4. [Selectors](#selectors)
5. [BEM](#bem)
6. [Architecture](#architecture)


## Formatting

For clarity and consistency:
* Indent with tabs. [So it is written, so let it be done](http://caplin.github.io/StyleGuide/).
* Opening brace on selector line.
* Closing brace on new line.
* A single space before #the opening bracket.
* A single space after the colon.

``` css
.class {
  property: value;
}
```

These rules simplify diffs by ensuring only one change per line:
* All declarations on separate lines.
* Multiple selectors on separate lines.

``` css
.selector1,
.selector2 {
  property: value;
  property: value;
}
```

The only exception is a class with one declaration which performs a specific function. This can be on a single line:

``` css
.red { color: red; }
```

Comments are preceded by a newline.
``` css
.class {

  // Hello World
  property: value;
}
```

**WRONG:**

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

## Whitespace

CSS is minified and stripped of whitespace and comments for production, so make *liberal* use of them.

Group related rules together and separate with newlines:

``` css
.selector {

  // Box model
  display: inline-block;
  margin: 0 auto;
  padding: 10px 20px;

  // Styling
  color: #FFF;
  background-color: #000;

  // Fonts
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

Separate different components with a block comment and 3 lines of whitespace:
``` css
/**
 * MyButtons
 * These are my buttons. Don't touch 'em.
 */
.MyButtons { ... }



/**
 * MyDivs
 */
.MyDivs { ... }
```




## Commenting

When looking at CSS, you cannot know:

* Where it is being used.
* What other CSS relies on it.
* What it relies on.
* What the intended effect is.
* What might break when you change it.

One defense is good commenting.

When using unusual properties:

``` html
<div class="container">
  <button />
  <button />
</div>
```

``` css
button {

  // Cleared by .container overflow
  float: left;

  // Uses box-sizing: border-box
  border: 2px;
  width: 50%;
}
```

When doing calculations:
``` css
// Height 50px
button {

  // Font + Padding + Border = 50px.
  font-size: 20px;
  padding: 14px;
  border: 1px;
}
```





## Selectors

### Intent

The selectors used when creating rulesets should match the *intent* behind the ruleset.

For example, when styling a new navigation (which happens to be in the footer):
``` css
// This will style every anchor in the footer, when the intent was only to style links in a secondary navigation.
footer a { ... }

// Better. The secondary nav can now be used elsewhere on the page.
.secondary-nav a { ... }

// Best. This does not select all anchors in the SecondaryNav, some of which may be used for other purposes.
.secondaryNav-link { ... }
```

``` html
<nav class="secondaryNav">
  <a class="secondaryNav-link" />
  <a class="secondaryNav-link" />
  <a class="secondaryNav-socialIcon" />
</nav>
```

> There are only two hard things in Computer Science: cache invalidation and naming things.

Classes should not describe the exact style being applied, as they cannot be changed:
``` css
.blue { color: blue; }

// After theming
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
.nav-link { color: blue; }
```

### Nesting
#### Try not to nest your CSS.

[*The descendant selector (whitespace).*](https://developer.mozilla.org/en-US/docs/Web/CSS/Descendant_selectors)

``` css
.btn { ... // Default styling }
.block button { ... }
```

``` html
<header class="block">
  <button class="btn" />
</header>
```

While nesting decreases the classes required in the HTML, the CSS is now dependent on the HTML structure.

Could this be rewritten to make it reusable? If it is a variant of a button, initially created for this location, how about creating a variant of the button class?

``` css
.btn { ... }
.btn--variantA { ... }
```

``` html
<header class="block">
  <button class="btn btn--variantA" />
</header>
```

#### Specificity
Specificity is increased when the descendant selector is used.
``` css
// This:
.block a { ... }

// Is more specific than this:
.block-link { ... }

// Now you have a specificity war with your own code, leading to either:
header .block-link
a.block-link { ... }
.block-link { ... !important }

// And the cycle will continue...
```


``` css
header .block-link // Can no longer be used outside header
a.block-link { ... } // Can no longer be used for non-anchors
.block-link { ... !important } // Can cause specificity problems if reused
```

#### Direct Child
The direct child selector `>` can be used to improve performance and limit the cascade of styles (good). But what if the DOM needs to be changed and another layer of HTML introduced? Now the CSS needs to be modified for what should be a trivial change.

For example, moving from this:

``` html
<header>
  <div class="logo" />
</header>
```

``` css
header > .logo { ... }
```

To this:
``` html
<header>
  <div class="grid-left">
    <div class="logo" />
  </div>
  <div class="grid-right">
    <div class="banner" />
  </div>
</header>
```

Will break your CSS.


### No ID's.
ID's are allowed for JavaScript and fragment identifiers, but should never be used in CSS.

* ID's override any styles applied by classes and then require the use of even more specific selectors to override the ID's rule (and so on).
* Only one ID can exist on a page, therefore the ruleset cannot be used more than once.
* JavaScript ID's cannot be changed or removed for fear of altering styling.

``` css
// The performance of these are virtually identical.
#block { ... }
.block { ... }
```

### No qualifiers
``` css
// No.
div.box { ... }
```

The `div` qualifier makes this more specific than any other class applied to the element.


### No nested wildcards
``` css
.block * { ... }
```

While the wildcard does not *always* have the significant performance impact it used to, there are very few cases where it is the correct course of action. 

[Non-nested wildcards can be used sparingly.](http://www.paulirish.com/2012/box-sizing-border-box-ftw)


### No reactive !important

!important is the nuclear option when it comes to specificity. If following the rules above, it should never need to be used reactively. That is, to override rule already being applied.

It does have a place when attempting to ensure a class will always work as expected. For example:
``` css
.hidden { display: none !important; }
```




<!-- ## General Principles

### Location Agnostic


### Minimal overrides


### Bad Abstraction


### DRY


### Open/Closed -->


## BEM
[BEM Methodology](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/).

BEM stands for Block Element Modifier. It looks like this:

``` html
<div class="Filter">
  <div class="Filter-row Filter-row--hightlight">
  </div>
  <div class="Filter-row">
  </div>
  <div class="Filter-row">
  </div>
</div>
```

A Block is a reusable piece of intertwined HTML and CSS. 
In this example, we are creating a *Filter*.

An Element is a child DOM element that forms an essential part of the component we are trying to build.

So `Filter-row` is an Element which will uniquely style rows inside `Filter` Blocks.

A Modifier is a variant of either of a Block or one of the Elements it contains.
In this example, we want the first row to stand out, hence the modifier of `--highlight`.


*The syntax defined here differs from the original BEM (see [SuitCSS](https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md)), but the end result is the same.*


### Block
The single root element for every Block.

The Block should be named with PascalCase (camelCase with a capitalised first letter).
Every selector related to the Block should be prepended by the Block name, this namespaces the Block and ensures styles do not leak or conflict with other parts of the page.
``` html
<div class="Filter"/>
```
``` css
.Filter {...}
```


### Elements
Elements within a Block should be named with camelCase and separated from the Block by a hyphen.
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

**Note: The BEM 'tree' does not replicate the DOM tree. Elements within Elements should *not* be named as such.**

``` html
<div class="Filter">
  <div class="Filter-outerElement">
    <div class="Filter-innerElement" />
  </div>
</div>
```
``` css
// Incorrect
.Filter-outerElement { ... }
.Filter-outerElement-innerElement { ... }

// Correct
.Filter-outerElement { ... }
.Filter-innerElement { ... }
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
  <div class="Filter-element--large" />
</div>
```
``` css
.Filter-element {...}
.Filter-element--large { height: 9001px; }
```

### States
States follow the same naming as Modifiers, but are prepended by `is-`.
``` html
<div class="Filter Filter--is-selected" />
```
``` css
.Filter {...}
.Filter--is-selected { border: red; }
```
States are a variation on modifiers - in the original BEM, there is no differentiation.

If the class is added or removed over the life of the element (via JavaScript), it is a State.

*Note: If there are several common States being used in an application, it may be wise to create a reusable set. For example:*

``` css
.is-hidden { display: none !important; }
```

*Care should be taken to keep these 'global' States reusable.*


### More Info
<!-- If the child element is not specifically tied to the Block, that is if it is , it should not be an Element. -->
A child element of a Block should only be a BEM Element (of that Block), if the element *should* only ever be used within that block.

<!-- In our Filter example, we have the `Filter-row` Element. If the `Filter-row` is used for styling unique to the Filter Block, then it is correct to make it a BEM Element.

``` html
<div class="Filter">
  <div class="Filter-row">
    <button />
    <button />
  </div>
</div>
```

However, if `Filter-row` was merely being used to clear the floats on the buttons, it should *not* be a BEM element. In this situation, a generic class (e.g. `.clearfix`) should be used/created.

``` html
<div class="Filter">
  <div class="clearfix">
    ...
  </div>
</div>
```

In the majority of cases, BEM Elements will need some generic rulesets *and* some rules specific to their function. Rather than duplicating rules in CSS, the element should be composed of several CSS classes.

``` html
<div class="Filter">
  <div class="Filter-row clearfix">
    ...
  </div>
</div>
```

Factoring out a reusable component from the CSS (in this case `.clearfix`) and leaving the styling specific to `Filter-row`, removes duplication and let's us reuse the `.clearfix` class.  -->

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

If the buttons are generic and used in multiple places, a generic class (e.g. `.button`) should be used:

``` html
<div class="Filter">
  <div class="Filter-row">
    <button class="button" />
    <button class="button" />
  </div>
</div>
```

The `.button` CSS should be flexible enough to allow the `button` class to be used in a variety of situations. Aka Location agnostic - not dependent on it's container

In the majority of cases, elements in a BEM Block will need some generic rulesets *and* some rules specific to their function. Rather than duplicating rules in CSS, the element can be 'composed' with several CSS classes.

``` html
<div class="Filter">
  <div class="Filter-row">
    <button class="button Filter-button" />
    <button class="button Filter-button" />
  </div>
</div>
```

Factoring out the reusable component from the CSS (in this case `.button`) and using `Filter-row` to apply button styling specifically for a `Filter` Block, removes duplication and let's us reuse the `button` class. 


*Note: Composing the style of an element with multiple classes is one of the main principles of [OOCSS](http://appendto.com/2014/04/oocss/). The other is the separation of structural and skin classes.*


## Architecture

All CSS at the Blade level should be BEM. The encapsulation provided by BEM means that no other elements can be affected by, or become dependent on, the CSS written in a particular blade.

Reusable classes should be created at the App or Bladeset level

If in doubt, package your CSS with BEM, rather than create generic classes.

```
[Expand]
[    Expand   ]
[       Expand      ]
```

## Recommended Reading

* [CSS Guidelines](http://cssguidelin.es/)
* [SuitCSS](https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md)
* [CSS Wizardry - BEM](http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/)
* [CSS Wizardry - Code Smells](http://csswizardry.com/2012/11/code-smells-in-css/)
