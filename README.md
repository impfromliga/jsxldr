# jsxldr
MicroLoader for fancy jsx components

0. [Deploy](#deploy) ([clone](#clone)/[install](#install)/[run](#run))
1. [Integration](#integration)
    - [Connect the library](#connect)
    - [Include youre components](#components)
2. [Component syntax &lt;html&gt;](#syntax)
    - [&lt;style&gt;](#style)
    - [&lt;script&gt;](#script)
3. [Lifecycle](#lifecycle)
   - [Component Constructor](#construct)
4. [$ as selector](#selector)
    - [Redirection (computing)](#computing)
    - [Сhaining selectors](#chaining)
5. [$ internal api (jsx supperclass)](#api)
    - [$.toString()](#tostring)
    - [$.removeChildren()](#removechildren)
    - [$.tree()](#tree)
6. [$ as jsx scope](#scope)
    - [S({})](#dto)
7. [Nested Components](#nested)
8. [Bind](#bind)
    - [events](#event)
    - [extend from child](#extend)
    - [emit to parent](#emit)
9. [Slots](#slots)
    - [main](#mainslot)

# <a id=deploy></a>Deploy
## clone
```shell
git clone https://github.com/impfromliga/jsxldr.git
```
## <a id=install></a>install
```shell
npm i
```
## <a id=run></a>run (Live Preview)
```shell
npm run start
```
# <a id=integration></a>Integration
## <a id=connect></a>Connect the library
- just ad the end of the body:
```html
<script type="module">import jsxldr from"./jsxldr.js";jsxldr()</script>
```
## <a id=components></a>include youre components
```html
<div _class="name:path/component.ext<constructorArgStaticString>"></div>
<div class="native" _class="path/autoname.htm"></div>
<?--same as-->
<div _class="autoname"></div>
```
- load jsx from file "path/component.ext" to &lt;div class="name"&gt;
- if exist native class, will do (".native").classList.add("autoname")
- can pass string to constructor by adding &lt;text&gt; (JSON passing soon)
- required only one lexeme is filename only
- file extension can be omitted, default was .htm
- omitted classname was filename without path and extension 
- _class names started with "__tpl\_" reserved for prefix late on parent loading
- see more about pass arguments to [Component Constructor](#construct)
# <a id=syntax></a>Component syntax
## &lt;html&gt;
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
- component can be writed as .html file

Compiled element will look like
```html
<div calss="native_class jsx_class" jsx=internal_jsx_id_number_autoincrement>
	<!-- youre layout -->
	<style>
    	<!-- youre style section -->
	</style>
</div>
```
- js is Evaled & scoped virtual
## <a id=style></a>&lt;style&gt; section
- every leading .__ pattern will replaced by root component className ( .foo in example)
- you can better combine styles by BEM syntax
## <a id=script></a>&lt;script&gt; section
- will be wrapped to
```js
function load($,arguments){
	//wrapping section code
}
```
- function runs when DOM is loaded
- this will applyed to rootElement ( .foo in example)
- $ pass the patched query selector functionality and above component scope
- arguments pass from instantiator

# <a id=lifecycle></a>Lifecycle $('@STAGE')
- init time is first time of running wrapped youre &lt;script&gt; section, so use top of it
```js
<script>
	//init
await $('@MOUNT') //resolved before parent loader(this):Promise
  //after mounted code...
await $('@DISMOUNT') //resolved before parent removeChilds(this):Promise
  //dismount for now only for root child (would be upgraded soon)
</script>
```
## <a id=construct></a>Construct syntax
as you see above the root level of the &lt;script&gt; section exactly is init

so access the construct arguments was the same as in native js function scope
- it allows you to reduce this calling, and you can set the variables direct.
All of them anyway will scoped. This shorthand thought to be quite acceptable for tini loader.
Whatever you type pure architecture with concise code of components it won't be hard to keep reduced scope clear too.
And just look how it simply clear:
```html
<script>
	console.log(arguments)
</script>
```

for pass arguments from instantiator add the trailing &lt;brackets&gt; in _class argument
```html
<div _class="name:path/component.htm<argument>"></div>
```
- for now it just as string, so you can use JSON for dynamic appending (parser will be standardized soon)

# <a id=selector></a>$('.query.selector')
## basic features:
will work also in &lt;style&gt; section
- selector for querySelector method will replace .__ leading pattern by root className (.foo in example)
- exact .__ selector will return this (rootElement) directly
- [An+B] or [B] will replace to :nth-of-type(An+B) A is 0 as default
- [An-B] or [-B] will replace to :nth-last-of-type(An+B) A is 0 as default
- [] at the and of css selector turn selectAll mode (must place before advanced $ query part)
## advanced:
that work in $() and _click= (etc events) attribute binds
- starting with '&lt;' selector will pass trailing string to parent jsx querySelector (eny times)
- starting with '&gt;' selector will replace as :scope&gt; (Selectors Level 4)
- trailing '$' will return jsx scope of finding element(s)
- after '$' allowing to follow prop1.prop2...propN chain that will get this from jsx scope
	- combining like '>$' will return first children (or select all in .removeChilds)
	- '<$' return the parrent jsx scope
	- '<$emit' return parrent.emit prop - in can use for binding event from arrtibute, or emit from js
## <a id=undefined></a>$(undefined)
dummy
- return Promise.all( this.loadChildrenNow() )
- shortcut for update & upload new inserted [_class] components
```js
//now can use for cheepy inline adds like:
$(
  this.insertAdjacentHTML('beforeend',`<div class=${$}__el_modifi></div>`),
  this.insertAdjacentHTML('beforeend',`<div class=${$}__el_modifi></div>`),
  this.insertAdjacentHTML('beforeend',`<div class=${$}__el_modifi></div>`),
).then((...$loadedChildren)=>{})
```
## <a id=computing></a>Redirection - $('selector', function):Promise&lt;[]&gt;
for map selector results by another function
```js
$('>.__el_mod', this.removeChild).then(([...mapped_result])=>{})
```

## <a id=running></a>running - $('selector', ...args):Promise&lt;[]&gt;
if selecting a function(s) will call it with args
```js
$('>.__el$method', 1,2,3).then(([...mapped_result])=>{})
$('>.__el$method<bindArg>')
//same as:
$('>.__el$method').bind(this,'bindArg')
```
- you can bind some static ars first of dynamic arg call
- and it will work in [Bind events](#event) binding
- and in [Component Constructor](#construct) args binding too

## <a id=chaining></a>Chaining $('selector', 'selector', ..., function):Promise&lt;[]&gt;
(in test)
- in this case every selector will recursively interpreate by current child
- ".__" pattern and "$" same individual

### Event-Selector-Handle:
```html
<a href="home"></a>
<a href="products"></a>
<a href="contacts"></a>
<script>
$('@MOUNT',`[href="${arguments}"]`,el=>el.className='on')
//this will await '@MOUNT' event
//then select by tamplated string applying the arguments passed from instantiator
//then natively set selected element className to 'on'
</script>
<style>.on{color:green}</style>
```
### Selector[]-EachHandle
you can select multiple, then pass the callback, and it would map each find elemen. The results was Promisyfi.all, so it can be async
```js
$('@DISMOUNT',`.__header, .__content, .__footer[]`, each=>each.value)
.then(console.log.bind(0,'Dismounted:'))

$('@DISMOUNT',`.__header, .__content, .__footer[]$`, $each=>$each.save() )
.then(console.log.bind(0,'All saving:')) //
```
# <a id=api></a>Internal $. api methods
(inherited from jsx supper class)
- this methods are reserved and dont iterable by for / forEach / Object.keys()
## <a id=toString></a>$.toString()
- return root component className.
- It usefull for pass root component className to string representations (like template strings binding) by passing component scope
```js
	`<div class=${$}__element_modificator>`
//	<div class=block__element_modificator>
```

## <a id=removechildren></a>$.removeChildren(el | el[] | String)
- remove element(s) from jsx by selector. Can use multiple by postfix []$
- dont throwing Exception, instead of this -
- return deleted elements[] or null if no one deleted (for youre self check)
## <a id=tree></a>$.tree(tabs:Number)
debug
- can be used for log recursive tree of virtual element binded scope (children & methods)

# <a id=scope></a>$ as jsx component scope
this can be redefine and/or add new ones
- there are simply set $.youre_prop = youre_value;
- there are public for eny other jsx componet witch have link to this scope
- used for dom events bindins by parent, child, or component itself
## <a id=dto></a>$({})
- get dto (soon)

# <a id=nested></a>Nested Components
- Worked by static (recursive fetching files)
- Worked by calling $(undefined) (or without argument)
	- then new :scope>[_class] elements will detected

# <a id=bind></a>Bind events
You can force config const eventTypes to extend library by other native DOM events, but some may need extra checks. For now config set supporting for following:
- click, dblclick
- change, input
- dragstart, drop
  - for now used manually <... ondragover=arguments[0].preventDefault()>

property getter in all case getters of $.property descriptor will run once on binding
## <a id=event></a>bind event to self method
```html
<button _click=$method></button>
<script>
	$.method=e=> //do something
</script>
```
## <a id=extend></a>extend call child method
```html
<div _class=sub:subcomponent.htm></div>
<button _click=.sub$method></button>
<button _click=".sub$method<Text>"></button>
```
- click will call $.method of .sub element from subcomponent.htm &lt;script&gt; section
- can bind arg first event will calling 
## <a id=emit></a>emit event to parent
```html
<button _click=<$method></button>
//with bind static argument "Text" first, before event will be called
<button _click=<$method<Text></button>
```
- emiting to parrent even more useful to prebind some child data

# <a id=slots></a>Slots
## <a id=mainslot></a>Main slot
You can add sub layaut inside jsx component, by default it will append at the begin of it
  - the jsx binds will applied as they'll be add by component layout
  - css injections will not be parsed by library features (except _class __tpl\_)
  - no js injections at all
```html
<div class=__template _class=jsx.htm>
    Task-list <button _click=<$create>+</button>:
</div>
```
 - add &lt;!--slot-main--&gt; for choose place for main slot (soon)
## <a id=tpl></a>Templates
You can bind some template or sub layouts by naming classes element (BEM) as '__tpl_name'
```html
<div _class=jsx.htm>
    <div class=__tpl_head>
        <h1>Header</h1>
    </div>
    <div _class=__tpl_foot:footer.htm></div>
</div>
```
- class & _class attr will prefix if it starts with __tpl\_ (ever if it add by parent layout)
- _class attr will also fetch sub component file, so it can be nested too
- names of templates provide by specified component
- [class*="__tpl\_"]{display:none} is global default recommend

# TODO Roadmap
## features
selector standartize:
- $({}) dto
- mixed chaining rules:
  - $(bool, ...) for assert feature. Reject can .catch(f=>)
  - $('@mousedown','@mousemove').then(dragHandle)
  - $('@PAUSE', '@RESUME', '@SUBSCRIBE', '@PIPE')

arguments standartize:
- parent &lt; selector at different position
- mixed chaining rules
  - '|' piping pattern - split the rules ('|=' will not separate)
  - `{q}|<$parentMethod`
- _event='<$parentMethod{\$}' - carring arg \$ //dont pass this
  - '$fn{arg}|<$parentMethod' - carring arg; on event call fn(arg,event); pipe
  - '$fn()|<$parentMethod' - on bind call fn; then look what it return, to do:
  - Promise '|<$parentMethod' - await Promise; then look what it return, to do:
  - Function '|<$parentMethod' - on event will call fn(event); then pipe
  - Bool '|<$parentMethod' - piping will active when bool assert
    - '$.prop|<$parentMethod' - active when prop assert
    - '$.assert()|<$parentMethod' - active if assert()
    - '@alt|<$parentMethod' - assert event when holding alt key
    - '@wait500|<$parentMathod' - event after timeout 500ms
    - '@move50|<$parentMathod' - after mousemove more then 50px
    - '@down|@move50|<$parentMathod' - after event, mouse down, move
  - Date '|<$parentMethod' - keep event arg; wait the date; then pipe
  - Array '|<$parentMethod' - map(pipe)
  - Object '|<$parentMethod' - just pipe //if need for other types cover it
- _class='className:jsx.htm(arguments)' - basic as string
  - 'jsx($handle:Emitter)' validate emiting interfaces
  - 'bool|className:jsx.htm(arguments)' - inline if templating
  - _class='className:jsx.htm<Type|Type2|...|TypeN>' - typification of polymorph children 
 
css
- :fisrt/last-of-type selectors converts to \[0/-1] \$.child (if \$ elem)

slots:named

## optimization
- option clearing comments from jsx
- fetch cache (for now just browser/server chache somehow)
  - deep instantiating constructor standart (single function with different calls)

## fixes
- localize css scope for global selectors
- FIX: $.removeChildren at DOM auto insert rand &lt;tbody&gt; parent
- FIX: scope garbage
- FIX: dismount child recurse
- FIX: finding patterns like [0] in not _dashedAttributes (done)
## moot features
- autoload dynamic children jsx by MutationObserver
```js
$('@customEvent').then(callback) //Promise form for once emitting
$('@customEvent', callback)		//callback form for continuous
$({}) //one pass set component scope
f[Symbol.toStringTag] //detect async function binding (and then update)
f=>Promise[] //runtime detect Promise or Promise[] for update when resolve

shortcut $('beforeend',`<div _class=component></div>`) by insertAdjacentHTML and $.update
	$('<div _class=component></div>') //capture exact load targets
		_class=component(args) //component constructor args
		_value={$prop} //argument reactive bind

rig, jet, fit,
gem, orb, ice,
imp, lox, erg,
```
