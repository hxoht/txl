[![development sponsored by voltra.co](https://img.shields.io/badge/Development%20sponsored%20by-Voltra.co-yellow.svg)](https://voltra.co/)

# SYNOPSIS
All your html templating needs in under 10 lines of JS.

# USAGE

## BASICS
The `Template` function takes a string and returns a function that takes your
template "locals".

```js
const txl = require('txl')
const template = txl('<h1>${name}</h1>')

console.log(template({ name: 'Glen' }))
console.log(template({ name: 'Henry' }))
```

```html
<h1>Glen</h1>
<h1>Henry</h1>
```

Because the `txl` function takes a string, you can read its contents from
another file — separation of concerns.

## ITERATION
The `${...}` part of a template string is an expression. So you can use any
expression (anything that results in a value), such as `1 + foo`, `map()`,
`filter`, etc.

#### index.js
```js
const txl = require('txl')
const fs = require('fs')

const t = txl(fs.readFileSync('./index.template'))

t({ list: [1,2,3] })
```

```html
<ul>
  ${ list.map(n => `<li>${n}</li>`).join('') }
</ul>
```

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
```

## MIXINS
You can pass anything into the "locals" for a template, including functions.
This means you can do mixins!

#### index.js

```js
const Template = require('txl')
const fs = require('fs')

const t = Template(fs.readFileSync('./index.template'))
const m = Template(fs.readFileSync('./mixin.template'))

t({ names: ['Glen', 'Henry'] li: m })
```

#### index.template

```html
<ul>
  ${ list.map(name => li({ name })).join('') }
</ul>
```

#### mixin.template

```html
<li class="name">${name}</li>
```

#### output
```html
<ul>
  <li class="name">Glen</li>
  <li class="name">Henry</li>
</ul>
```

## PLUGINS, FILTERS (CONDITIONALS)
Let's say you want to show a block of content based on a condition, you could
do this with short-circuit evaluation in some cases, or you could make a simple
plugin! A plugin is just a function...

```js
const txl = require('txl')
const template = txl(fs.readFileSync('./index.template'))

// a function that takes a boolean value and a string.
const IF = (b, s) => b ? s : ''

// pass in our values and our plugins
template({ foo: 100, IF })
```

```html
<section>
  ${ IF(foo > 100, `
    <div> This is only rendered if foo is equal to 10.
    </div>
  `) }
</section>
```

```html
<section>
</section>
```

## DOM DIFFING & PATCHING
Generally, you should be able to target nodes directly by using the
`querySelector` method. In some cases, you may feel like applying a
large change-set to a node. In this case, you could use [morphdom][1]!

```js
const morph = require('morphdom')
const txl = requre('txl')

const template = txl('<div>${count}</div>')
const someNode = document.querySelector('.some-node')

let n = 10

while (n--) {
  morph(someNode, template({ count: n }))
}
```

## SECURITY
You may be concerned with escaping depending on where your data comes from
or how it is stored. In this case there are a lot of really nice modules. For
instance [he][2] is a well maintained and very robust encoder/decoder.

[1]:https://github.com/patrick-steele-idem/morphdom
[2]:https://github.com/mathiasbynens/he
