---
layout: post
pagination: 
  enabled: true
title: Image etc. with caption
date: 2019-12-31
categories: learning
#draft: true
---

# figure with caption

<figure>
    <figcaption>Header</figcaption>
    <img src="/assets/images/DeltaMushPar3_CaseStudy_ANPinlining.png"
         alt="Elephant at sunset">
    <figcaption>Bottom caption</figcaption>
</figure>


## Usage notes
```
Usually a <figure> is an image, illustration, diagram,
 code snippet, etc., that is referenced in the main flow 
 of a document, but that can be moved to another part of 
 the document or to an appendix without affecting the 
 main flow.
Being a sectioning root, the outline of the content of 
the <figure> element is excluded from the main outline of 
the document.

A caption can be associated with the <figure> element by 
inserting a <figcaption> inside it (as the first or the 
last child). The first <figcaption> element found in the 
figure is presented as the figure's caption.

```


