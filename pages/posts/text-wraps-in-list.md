---
layout: base.njk
title: Very specific post about text wrapping in styled list
date: 2021-09-14
tags: post
---
# {{ title }}

### Styling your list

There are many fine instructions out there telling how to set custom bullet points for your list elements. In most cases it boils down to following

``` css
ul {
  list-style-type: none;
}

li:before {
  content: "🍸";
}
```

Add some positioning on top of that and it will work fine for most of the cases. 

### Rare case

Let's say we want to add word breaks there to display our list in narrow or flexible container

``` css
ul {
  list-style-type: none;
  word-break: break-word; //<---
}

li:before {
  content: "🍸";
}
```

In that case there is one corner case that is not covered. When there is a word that is too long to be placed in the list item, there will be unexpected line break between custom bullet and the word. 

[Demonstration on Codepen](https://codepen.io/totally___normal/pen/KKqXVPL?editors=1100)

This happens because `:before` is not exact replacement for native list marker (they even call it `::marker`). The content participates in [page layout flow](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/In_Flow_and_Out_of_Flow) and allows browser to see an opportunity to do a line break after it. They call it [soft wrap opportunity](https://drafts.csswg.org/css-text/#line-break-details).

### Removing :before from layout flow

There are at least two ways of excluding an item. According to [mdn article](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flow_Layout/In_Flow_and_Out_of_Flow#taking_an_item_out_of_flow) we can either use absolute positioning or float to do that. I find float to be pretty reliable in this case and recommend to use it. And in nearby future we can even use `inline-start` value so it works well with rtl languages. 

``` css
ul {
  list-style-type: none;
  word-break: break-word;
}

li:before {
  content: "🍸";
  float: left;
}
```

[Demonstration on Codepen](https://codepen.io/totally___normal/pen/JjJrRRN?editors=0100)

### What's next?

Of course this is a pretty hacky way to fix things. Ideal solution is to style native list marker to suit our needs and don't use `:before` at all. This works well, but unfortunately it is not yet widely supported in modern browsers.

``` css
ul {
  word-break: break-word;
}

li::marker {
  content: "🍸";
}
```

[Demonstration on Codepen](https://codepen.io/totally___normal/pen/wverzrZ?editors=0100)