# jsxLdr
microLoader for tiny jsx components

# Install
```shell
npm i
```

# Run (Live Preview)
```shell
npm run start
```

# Connect the library
- just add in head or in body:
```html
<script src="jsxLdr.js"></script>
```

# Include Components
```html
<div class="foo" jsxLdr="component"></div>
```
- deafault file extension is .htm it can be omited

# Components syntax
- component can be writed as .htm file

```html
<!-- component layout without root (omitted) -->

<script>
	/* component code (single section will be cut from src) */
</script>

<style>
	/* component styles (single section will be cut from src) */
</style>
```
- everything that is not cut will stay as html layout

## &lt;style&gt; section
- every leading .__ pattern will replaced by root className ( .foo in example)
- you can better combine styles by BEM syntax

## &lt;script&gt; section
- will be wrapped to

```js
function load($){
	//wrapping section code
}
```
- function runs when DOM is loaded
- this will applyed to rootElement ( .foo in example)
- $ argument pass the query selector functionality and above

### $(selector)
- selector for querySelector method will replace .__ leading pattern by root class (.foo in example)
- exact .__ selector will return this (rootElement) directly

### $.toString()
- return rootElement.className
- in string representations return root className (like template strings binding)
```js
	`<div class=${$}__element_modificator>`
//	<div class=block__element_modificator>
```

### $.removeChild(HtmlElement)
- just alias for this.removeChild

### $.appendChild(InnerHTML)
- overload this.appendChild(InnerHTML:string):HtmlElement|HtmlElement[]
- can append multiple, in this case return array of child nodes

### $.innerHTML
- setter and getter this.innerHTML with run $.load() after set
- innerHtml rebilds DOM, so it need hook to reload code section after, so that it is
- can be slow, for better choise see $.appendChild()

### $.load(js?)
- re/load the code section
- first time run automaticly onload when jsxLdr.js find elements with [jsxLdr] argument
- repeat calls can ommit js argument (pass last one)
- for now is appendix API

# Nested Components
- Worked by static
- Worked by $.appendChild(InnerHTML:String) call