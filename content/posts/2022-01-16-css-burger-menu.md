---
title: "Pure HTML & CSS \"hamburger menu\""
date: 2022-01-16T08:56:56Z
draft: false
comments: true
---
Over the last few days, I set up a website for my synth nerdery & music
production ramblings, [depthbuffer.uk](https://www.depthbuffer.uk/). One thing
I did with it that I think was quite fun was make the whole thing responsive
without using any Javascript, including replacing the menu column on the
right-hand side with a collapsible "hamburger" menu in the title bar when
viewed at low width.

Similar to this site, the new one is built using [Hugo](https://gohugo.io/) and
hosted on [Vercel's](https://vercel.com/) free "hobby" tier, with source and
assets available on
[GitHub](https://github.com/mangobrain/depthbuffer-uk-hugo). If you just want
to dive straight in to the code, the HTML lives in
[the partial template `header.html`](https://github.com/mangobrain/depthbuffer-uk-hugo/blob/main/layouts/partials/header.html),
with CSS
[here](https://github.com/mangobrain/depthbuffer-uk-hugo/blob/main/assets/css/main.scss)
(there's a lot in there, but it does have comments here & there).

NB: This might be a complete accessibility nightmare. I don't really know how
to check, let alone fix it if it is. Use at your own risk.

I've technically put the contents of that repo under a Creative Commons
[BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) license, but
I promise I won't come after anyone who cribs from this article or accompanying
source (which, realistically, will probably just be my own future self). It's
more the media assets I'm bothered about.

So, how does it work? Well, there are a few tricks used.

## Hiding the menu on wide devices

```scss
@media only screen and (min-width: 769px)
{
    nav.menu
    {
        display: none;
    }
}
```

(NB: I'm using Hugo's [ToCSS pipe](https://gohugo.io/hugo-pipes/scss-sass/) to
support SASS/SCSS, hence the non-standard features like nesting of selectors.)

When viewed in a viewport at least 769 pixels wide, hide the entire `nav`
element and all its children. This is an example of "mobile-first" responsive
design: the mobile behaviour is the default, with overrides for larger screens.

Because of this, the hamburger menu also triggers in desktop browsers, if you
make your window narrow enough. Try it!

## Invisible checkboxes for fun & profit

You may notice that in the HTML, there's a checkbox; but no checkbox appears on
the final page.

```html
<nav class="menu">
    <input type="checkbox" class="menu-open" id="menu-open"/>
    <label class="menu-open-button" for="menu-open">
        ...
    </label>
    <ul>
        ...
    </ul>
</nav>
```

```scss
nav.menu
{
    input.menu-open
    {
        display: none;
    }
}
```

This checkbox isn't only invisible on wide viewports, it's just flat-out
invisible. It serves two purposes:

1. Even though it's not visually displayed, its state can still be toggled by
   other (non-Javascript) means: specifically, clicking on a `<label>` element
   associated with the checkbox will toggle it.
2. When checked, the CSS selector `:checked` can be used to select it, allowing
   us to assign different styles to surrounding elements.

### Annoying your siblings

By combining the `:checked` and `~` (sibling) selectors, we can influence the
styles of other elements inside the same `<nav>` as the checkbox, not just the
checkbox itself.

```scss
nav.menu
{
    // Hide unordered lists that are sibilings of the menu-open checkbox
    input.menu-open ~ ul
    {
        display: none;
    }

    // When the checkbox is checked, show them
    input.menu-open:checked ~ ul
    {
        display: block;
    }
}
```

This, fundamentally, is what ties the whole thing together. The snippet above
covers showing & hiding the unordered list that makes up the body of the menu.

## You said there'd be burgers!

To show the menu, you click on the label, and the content magically appears.
So far we haven't actually put anything interesting inside the label, but this
is where the actual "hamburger" in the "hamburger menu" lives; it's entirely
formed of child elements inside the label. As long as those elements are
styled using CSS, and the properties you want to animate when clicked can have
[CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions)
applied, then you can use the same trick as above to trigger the animation:
put the initial state in the "base" style, and the final state in the
`:checked` style.

I happen to be using two trios of `<span>` elements to make the burger itself:
three with a solid white background, and another three behind them with
transparent backgrounds & drop shadows. I do this because I like drop shadows,
but in the final state of the animation, two of the white spans overlap each
other; so if the shadows were attached directly to the white spans, one would
be partially in shadow when the menu is open, instead of neatly forming a cross
with a shadow.

```html
<nav class="menu">
    <input type="checkbox" class="menu-open" id="menu-open"/>
    <label class="menu-open-button" for="menu-open">
        <span class="burger-1"></span>
        <span class="burger-2"></span>
        <span class="burger-3"></span>
        <span class="shadow burger-1"></span>
        <span class="shadow burger-2"></span>
        <span class="shadow burger-3"></span>
    </label>
    <ul>
        ...
    </ul>
</nav>
```

```scss
nav.menu
{
    // Normal style & initial state of any animated properties.
    // We are selecting the label with class "menu-open-button" which is a
    // sibling of the checkbox with class "menu-open".
    input.menu-open ~ label.menu-open-button
    {
        // Fixed width & height - used as the basis for sizing and
        // positioning the spans, which use tricks like negative margins
        // so as not to take up space in the document flow.
        position: relative;
        display: block;
        width: 14px;
        height: 14px;

        span
        {
            // Starting "untransformed" position of all six spans
            // is right in the middle of the label.
            position: absolute;
            top: 50%;
            left: 50%;
            width: 14px;
            height: 2px;
            margin-left: -7px;
            margin-top: 1px;

            // When CSS transformation changes, animate the transition
            transition: transform 200ms;

            // White bars on top
            z-index: 2;
        }

        span.shadow
        {
            // Shadows underneath
            z-index: 1;
        }

        // Apply CSS transformations to each span individually, to put them
        // in their starting positions.
        span.burger-1
        {
            transform: translateY(-4px);
        }
    }

    // When the menu is open (i.e. the checkbox is checked), change CSS
    // transformations on each span to put them in their finishing positions.
    input.menu-open:checked ~ label.menu-open-button
    {
        span.burger-1
        {
            // ...
        }
        // ...
    }
}
```

Are there better ways of doing this? Probably, yes. But visually, this achieves
the desired end result, without mucking around with SVG, animated GIFs, or
JavaScript.

## Lettuce & Pickles

A couple of little touches to finish it off. As mentioned earlier, the menu is
also usable in desktop browsers, when resized to be narrow, so it'd be nice if
the mouse cursor changed to indicate that the burger is clickable. Also, in
Chrome on Android, a blue highlight appears over the burger when tapped, which
someone spoils the effect; we can hide this with a vendor-specific extension.

```scss
nav.menu
{
    input.menu-open ~ label.menu-open-button
    {
        cursor: pointer;

        // NB: "none" doesn't work here; it seems it has to be set to an
        // actual colour, but with an alpha value of 0.
        -webkit-tap-highlight-color: rgba(0,0,0,0);
    }
}
```

## Fin.

Why go to all this trouble, you might ask, then leave the menu itself as a
boring, non-animated block that just appears when clicked? Well, I would have
liked to have a vertical slide out animation, but as far as I know you can't
currently create one without JavaScript. In particular,
[you can't apply CSS transitions to `auto` dimensions](https://css-tricks.com/using-css-transitions-auto-dimensions/).
Yes, I know jQuery _et al._ probably already have handly helper functions for
this, but as I've got this far without using any JavaScript at all, I'd rather
keep it that way.
