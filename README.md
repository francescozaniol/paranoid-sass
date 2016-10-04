##PARANOID SASS

Paranoid SASS is an approach to SASS that protects the global style from unexperienced devs, and avoid CSS leaking as much as possible.

This approach can be used when:
- Using BEM-like syntax
- Dealing with large codebase
- Dealing with a team of unexperienced FE devs (or simply BE devs trying to write FE code)
- Not having much time to continuously review code written by others

###Background

The idea is an outcome of my struggling by being the lead fronend dev for a large scale website and having to deal with teammates who would always come out with new ways to mess up the code.

###Standard approach

The following example shows the standard approach when using BEM-like syntax:

The HTML:

```html
<div class="block">
    <p class="block__description">Hi</p>
    <p class="block__description--highlighted">You're cool <span class="fa fa-thumbs-up" aria-hidden="true"></span></p>
</div>
```

The SASS for "main.scss":
```sass
@import "global/typography";
@import "global/helpers";

@import "components/block";
@import "components/block-2";
@import "components/block-3";
// ...
```

The SASS for "_block.scss":
```sass
.block{
    margin:1em;
    &__description{
        padding-bottom:1em;
    }
    &__description--highlighted{
        background:red;
    }
}
```

The standard approach is safe and clean if you know what you're doing. However, if dealing with unexperienced devs can lead to major CSS leaking.

####What could go wrong with the standard approach?

Consider the previous example. One day, I've asked a BE dev to change the color of the font-awesome icon. That's what he wrote:

```sass
.block{
    margin:1em;
    &__description{
        padding-bottom:1em;
    }
    &__description--highlighted{
        background:red;
    }
}

.fa{
    color:white;
}
```

Can you see what's wrong here? The root issue is that the global scope is always exposed (therefore in danger) for any SASS partial of your codebase.
On top of this, many devs just forget to write class names using the BEM syntax, the outcome is that I need to send the code back to be refactored.

###The paranoid approach

The paranoid approach:
- Forces devs to use the BEM convention when writing HTML
- Protects the global style in selected SASS partials
- Limits (as much as possible) CSS leaking

The SASS for "main.scss":
```sass
@import "global/typography";
@import "global/helpers";

block__{ @at-root{ @import "components/block"; }} // <--
block-2__{ @at-root{ @import "components/block-2"; }} // <--
block-3__{ @at-root{ @import "components/block-3"; }} // <--
// ...

```

The SASS for "_block.scss":
```sass
.#{&}description{
    padding-bottom:1em;
}
.#{&}description--highlighted{
    background:red;
}
```

The compiled CSS:
```css
.block__description{
    padding-bottom:1em;
}
.block__description--highlighted{
    background:red;
}
```

####Why this approach?

As you can see the blocks' "@import"s are *nested* so any selector inside "_block.scss" MUST start with ".#{&}". If not, the generated CSS simply doesn't work. For example, if in "_block.scss" we include:

```sass
.fa{
    color:white;
}
```

The compiled CSS would be:
```css
block__ .fa{
    color:white;
}
```
Which doesn't match anything.

Also, escaping the whole file with "block__" (and not just ".block") forces devs to use the BEM syntax when writing HTML.

####Few important notes

- If you want to style the root container (".block") I suggest assigning the additional class "block__root":
```html
<div class="block block__root">
    ...
</div>
```

So "_block.scss" can contain:
```sass
.#{&}root{
    margin:1em;
}
```

- The selector "block__" is used instead of ".block__" cause it allows to specify, if needed, IDs:
```sass
#\#{&}unique-id{
    display:block;
}
```

###Take the paranoia to the next level

In order to lower CSS leaking and bugs even more, I would set a Git-Enforced-Policy to any SASS file that deals with the global scope. The enforced policy would reject any push to "main.scss" or any of the SASS partials inside "./global" so that only experienced devs will be able to push changes that affect the global space.
Unfortunately I haven't had the chance to implement the enforced policy with my team; for more info see here: https://git-scm.com/book/en/v2/Customizing-Git-An-Example-Git-Enforced-Policy.

###Make it impossible to have css leaks (work-in-progress)

CSS leaks can still happen if different components are nested one inside another. For example if "_block.scss" contains:

```sass
.#{&}root{
    .fa{
         color:white;
    }
}
```

The global scope is safe cause only ".fa" inside the block are styled, but what if another component having a ".fa" is nested inside the block?

```html
<div class="block__root">
    ...
    <span class="fa fa-thumbs-up" aria-hidden="true"></span>
    ...
    <div class="block-2__root">
        ...
        <span class="fa fa-thumbs-up" aria-hidden="true"></span>
        ...
    </div>
</div>
```

This could be avoided by, for example, forcing only a 1-level nesting, or removing any "not-bemmed" selectors.
I tried to achieve this in SASS but could not work it out. Maybe a PostCSS plugin could do the trick?
