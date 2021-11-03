---
title: How CSS is Parsed
date: 2019-02-08 16:40:15
tags: CSS
categories: CSS
---
Three Parts  
### Cascade and Specificity
Cascade: process of combining different stylesheets and resolving conflicts between different CSS rules and declarations, when more than one rule applies to a certain element.

1.Cascade Priority:  
`Importance -> Specificity -> Source Order`
{%asset_image cascade.png%}

2.Summary:  
* CSS declarations marked with !important have the highest priority.(only use it as a last method).  
* Inline styles > exteral stylesheets style.  
* The universal selector * has no specificity value(0, 0, 0, 0).  
* When same importance and specificity, the last declaration will override all other declarations.
* Rely more on specificity than on the order of selectors(because order changes more easily).  
* Always put the 3rd-party stylesheets before your own stylesheets.

3.Example
```html
<nav id="nav">
  <div class="pull-right">
    <a class="button button-danger"
       href="link.html">Don't click here!</a>
  </div>
</nav>
```

```css
body {
  padding: 50px;
}

.button {
  font-size: 20px;
  color: white;
  background-color: blue;
}

a {
  background-color: purple;
}

#nav div.pull-right a.button {
  background-color: red;
}

#nav a.button:hover {
  background-color: yellow;
}
```
The button background color is red.  
But if add "! important" at the end of any color, like:
```css
a {
  background-color: purple !important;
}
```
The bg color is purple. Except for the a.button:hover, because it changes color only if hover.


If cursor hover the button, the color doesn't change, because the precedency is lower than its top one. Solve it by changing to:
```css
#nav div.pull-right a.button:hover{
  background-color: green;
}
```
now when hover, the color is green.

### Value Processing
Each property has a inital value.  
1.The order of value in the processing  
declared value, cascaded value(after the cascade), specified value(default value, if there is no cascaded value), computed value(convert relative value to absolute, eg. 1.5rem -> xx px), Used value(final calculation based on layout), Actual value(browser and device restriction, round the decimal)

2.The unit computation  
{%asset_img unitexample.png The Unit Computation %}


### Inheritance
* Not all the properties has the inheritance.
* Inheritance only works if there is no value for the property.
* The 'inherit' keyword forces inheritance on a certain property.Eg. box-sizing:inheritance;
* It is computed value that passed from parent to child, not declared value.

