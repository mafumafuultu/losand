# \_(losand).\_
## \_(losand).\_ is a monad imprementation on any Javascript
[![Maintainability](https://api.codeclimate.com/v1/badges/bb24ccbb4103dec4837f/maintainability)](https://codeclimate.com/github/johnny-shaman/losand/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/bb24ccbb4103dec4837f/test_coverage)](https://codeclimate.com/github/johnny-shaman/losand/test_coverage)
[![Build Status](https://travis-ci.org/johnny-shaman/losand.svg?branch=master)](https://travis-ci.org/johnny-shaman/losand)
## Usage

### node
~~~bash
npm install losand
~~~

~~~javascript
const _ = require("losand")
~~~
### worker
ex. inside ./lib subfolder
~~~javascript
importScripts("./lib/losand.js")
~~~
### browser
~~~html
<script src="https://cdn.jsdelivr.net/npm/losand@1.5.0/losand.js"></script>
~~~

~~~javascript
//join
_({a: 1}).join.a === 1
_({a: 1})._.a === 1

//map f(t1, t2)
const t1 = {a: 1}
const t2 = {b: 2}

_(t1).map(Object.assign, t2)._ === t1
t1.b === 2

//fit f(t2, t1)
_(t1).fit(Object.assign, t2)._ === t2
t2.a === 1


/* Monad Rows */
const a = {a: 1}
const k = x => _({a: x.a + 5})
const h = x => _({a: x.a * x.a})
const m = x => _(x)

/*
** Left Identity
*/

// map <- bind
_(a).bind(k)._.a === k(a)._.a
// fit <- link
_(a).link(k)._.a === k(a)._.a


/*
** Right Identity
*/

// map <- bind
m(a).bind(_)._.a === m(a)._.a
// fit <- link
m(a).link(_)._.a === m(a)._.a


/*
** Associativity
*/

// map <- bind
m(a).bind(x => k(x).bind(h))._.a === m(a).bind(k).bind(h)._.a
// fit <- link
m(a).link(x => k(x).link(h))._.a === m(a).link(k).link(h)._.a

// cash
_(3).map(n => n += 5)._  // 8
_(3).map(n => n += 5).re._ // 3

_({b: 1}).map(o => o.b += 1)._  // 2
_({b: 1}).map(o => o.b += 1).re._ // {b: 2}

// $ methods Chainable method
// ex1.
_({
  b: 0,
  add (v) {this.b += v}
})
.$($ => $.add(5))
.$($ => $.add(5))
._.b === 10

// ex2. (prototype chain's method can work)
_(
  Object.create({
		add (v) {this.b += v}
	},
	{
		b:{
			writable: true,
			value: 0
		}
	})
)
.$($ => $.add(5))
.$($ => $.add(5))
._.b === 10

// $$ have been chainable IO
_(
	Object.create({
		add (v) {this.b += v}
	},{
		b:{
			writable: true,
			value: 0
		}
	})
)
.$$((v, $) => $.add(v), 5)
.$$((v, $) => $.add(v), 5)
._.b === 10

//by return constructor
_({a: 1}).by._ === Object
_([1, 2]).by._ === Array

//is check type and after return it
_({a: 1}).is(Object)._ // {a: 1}
_({a: 1}).is(Array)._ // undefined
_({a: 1}).is(Array).re._ // {a: 1}

//isnt mismatch type and after return it
_({a: 1}).isnt(Array)._ // {a: 1}
_({a: 1}).isnt(Object)._ // undefined
_({a: 1}).isnt(Object).re._ // {a: 1}

//be is a passing test function
_(3).be(v => v === 3)._ === 3
_(3).be(v => v !== 3)._ === undefined

//fullen check all value isn't undefined or null
_({a: 3, b: undefined}).fullen === false
_({a: null, b: 4}).fullen === false
_({a: 3, b: 4}).fullen === true
_([undefined, 2]).fullen === false
_([4, null]).fullen === false
_([4, 2]).fullen === true

//keys
_({r: 128}).keys._[0] === "r"

//vals
_({r: 128}).vals._[0] === 128

//get
_({a: {b: {c: 3}}}).get("a", "b", "c")._ === 3
_({a: {b: {c: 3}}}).get("c", "b", "a")._ === undefined

//set
_({a: {b: {c: 3}}}).set(8, "a", "b", "c")._ // {a: {b: {c: 8}}}
_({}).set(4, "c", "b", "a")._ // {c: {b: {a: 4}}}

//been present methods and properties settings on wrapped Object's
const BT = _(function (v) {
  this.v = v;
})
.affix({
  a (v) {
    this.v += v;
    return this.v;
  },
  b (v) {
    this.v += v;
    return this;
  },
  c (v) {
    this.v += v;
  },
})._

_(new BT(3)).been.a(3).b(3).c(3)._ === 12
_(new BT(3)).been.a(3).b(3).c(3).to     // return to _(_(new BT(3)).been.a(3).b(3).c(3)._)
_(new BT(3)).been.v(5).d(3, "e").e(3)._ // BT{v: 5, d: {e: 3}, e: 3}

//draw is wrapped Object assign from argumentate Object
_({a: 3}).draw({b: 5}).$($ => {
  $.a === 3 // {a: 3} is keeping
  $.b === 5
})

//cast is wrapped Object assign to argumentate Object
_({a: 3}).cast({b: 5}).$($ => {
  $.a === 3 
  $.b === 5 // {b: 5} is keeping
})

//hold on wrapped Object from arguments and other object have same inheritance
_({a: 13, b: 24, c: 51, d: 40}).hold("a", "c", "f")._ // {a: 13, c: 51}

//crop out of wrapped Object from arguments and other object have same inheritance
_({a: 13, b: 24, c: 51, d: 40}).crop("a", "c", "f")._ // {b: 24, d: 40}

//pick up wrapped Array to argument Object and other object have same inheritance
_(["a", "c", "f"]).pick({a: 13, b: 24, c: 51, d: 40})._ // {a: 13, c: 51}

//drop out wrapped Array from argument Object and other object have same inheritance
_(["a", "c", "f"]).drop({a: 13, b: 24, c: 51, d: 40})._ // {b: 24, d: 40}

//turn is turn to key value pairing
_(["a", "b", "c"]).turn._ // {a: 0, b: 1, c: 2}

/*
relate is pairing two Objects but each Objects have no property and no inheritance and
swap is call _().relate paired Object another one
*/
_({c: 14}).relate({d: 23}).swap._.c === undefined
_({c: 14}).relate({d: 23}).swap._.d === 23
_({c: 14}).relate({d: 23}).swap.swap_.c === 14
_({c: 14}).relate({d: 23}).swap.swap_.d === undefined

_({c: 14}).relate(t1)
_(t1).swap._.c === 14

//list 
_({2: 24, 0: 1, 1: 35}).list // [1, 35, 24]

//define is Object.defineProperties on wrapped Object
_({a: 68}).define({
  defined: {
    configurable: true,
    value: 123
  }
})._.defined === 123



//create is create a new Object inherit on wrapped Object
Object.getPrototypeOf(
  _(a).create({})._ // Object.create(a, {})
) === a

//other is create same inherit Object
_(a).other({})._.constructor.prototype === a.constructor.prototype

//depend is create a new Object inherit on argumentate Object
Object.getPrototypeOf(
  _({}).depend(a)._
) === a

//.give is a iterator of wrapped Object to apply on argumentate function
_({a : 1}).give((key, val) => {
  console.log(key + ": " + val)
})

//json get JSONString
_({a: 1}).json === JSON.stringify({a: 1})

//[].each
[2, 3, 4].each((v, k) => console.log(k * v))

//[].aMap is ApplicativeMap method on Both Way
[v => v + 2, v => v * 2].aMap([1, 2, 3]) // [3, 4, 5, 2, 4, 6]
[1, 2, 3].aMap([v => v + 2, v => v * 2]) // [3, 4, 5, 2, 4, 6]

//[].adapt fill gapes like Array.prototype.shift()
[2, 3, 4, undefined, 6, null].adapt(5, 7) // [2, 3, 4, 5, 6, 7]

//[].adaptRight fill gapes like Array.prototype.pop()
[2, 3, 4, undefined, 6, null].adaptRight(7, 5) // [2, 3, 4, 5, 6, 7]

//Generate Sequence Array
//x._(lim, step = 1) likely [x ... lim]
2 ._(12);
2 ._(26, 3);

//String.prototype.json get _(parsed_object)
_({a: 3}).json.json._.a === 3

//.from map in prototype
_(Object).from._ === Object.prototype

//.affix write up prototype object on constructor
_(function () {}).affix({
  a : 3
}).from._.a === 3

//.annex define properties mapping prototype object on constructor
_(function () {}).annex({
  b : {
    configurable: true,
    value: 5
  }
}).from._.b === 5

//.fork inherit on constructor
Object.getPrototypeOf(_(Array).fork(function () {
  Array.call(this);
}).from._) === Array.prototype;

//.hybrid intterupt inhert on constructor
Object.getPrototypeOf(
  _(function () {})
  .fork(function () {
    Array.call(this);
  })
  .hybrid(Array.prototype).from._
) === Array.prototype;

//.part is patial applying
_((x, y, z) => (x + y) * z).part(null, 2, undefined)(3)(4)._
===
_((x, y, z) => (x + y) * z).part(null, 2, undefined)(3, 4)._
===
_((x, y, z) => (x + y) * z).part(3, 2, 4)._
===
((x, y, z) => (x + y) * z)(3, 2, 4)._

//.cache presence delay and force
_((x, y, z) => (x + y) * z).done(3, 2, 4).re.done(5, 3, 7)._ === 20

//.recache presence delay and force mutate cached value
_((x, y, z) => (x + y) * z).done(3, 2, 4).re.redo(5, 3, 7)._ === 56

//.apply applying functions
_(10).apply(v => v + 10, v => v * 3)._ === 60

~~~
## Event driven development

~~~javascript
const EETest = _(EventEmitter).fork(function () {
  EventEmitter.call(this)
})._

const eeTest = new EETest()

//addLitener
_(eeTest).on({
  a: 3,
  "get" () {
    this.a === 3 // true
    this.put() // can call it
  },
  "put": () => {},
  "post": () => {},
  "delete": () => {}
})

//addOnce
_(eeTest).once({
  b: 10,
  "get" () {
    this.b === 10 // true
    this.put() // can call it
  },
  "put": () => {},
  "post": () => {},
  "delete": () => {}
})

//Example with Promise

const promise = new Promise((res, rej) => {
  res({});
})
.then(v => _(v))
.then(v => v.draw(v => ({foo: foo(v)}))
.then(v => v.map(t => t.foo));
~~~