# Post
- - - -
title:console.tap
published: false
description: 
tags: 
- - - -

# Why We Make Room For `console.log` 

## Why are any of us here?
### To fix what ain't broke!

I have been programming for about 10 years. I spent the first four pretending to learn Java in college and then started with JavaScript in 2014. That was a little before Babel and the proposals for ES6 were introduced. I have gotten to see what JavaScript had been, the promise of what it would become, and now to see it surpass all expectation. Modern JavaScript, the evolution of the language starting with ES6, has been incredible but it has left a pinprick of a problem. 

### The Ktchn Snk 

Bellow is an example function that is using a slew modern JS features:

```
{% runkit
 
const isEqual = require('lodash.isequal');
const moment = require('moment');

function formatCurrency(num) {
    return (
        '$' +
        num.toString().replace(/(\d\d\d(?=[^.]))/g, function($1) {
            return $1 + ',';
        })
    );
}
const transactionData = {
    amount: 100,
    date: Date.now(),
    description: 'Shrimp! Heaven! Now!',
    type: '',
    from: 'Daniel',
    to: 'Mom'
};
%}
const pickAndFormatTransaction = ( {
        amount,
        date,
        description,
} ) => ( {
        amount: Number( amount )
            ? formatCurrency( amount ) 
            : amount,
        date: moment( date ).format( 'DD/MM/YYY' ),
        description
} );

console.log(pickAndFormatTransaction(transactionData));
{% endrunkit %}
```
<span></span><figcaption>❓What do you think this function would have looked like <a href="https://codesandbox.io/s/compassionate-fire-o6xnd?fontsize=14&view=preview">just a few years ago?</a></figcaption>

The function is concise and deliberate. By which I mean there is no extra syntax. I don't need to use a `return`, which also means I don't need to declare a `const resultingObject =...;` to return. I dodged having to name the parameter just to pull out some values. I also don't need to create an `if/else` block for `amount`. I don't mean to diminish things like `return` or explicitly naming values. They have their place, instead, I relish in how sparse the function can be, while still being understandable. In my eagerness to cut down on clutter, I have created a new hurdle.
 
The `pickAndFormatTransaction` is so concise, there's no room for a `console.log`. If something goes wrong it’s going to take extra work to debug it.

### Something Goes Wrong

```
{% runkit
 
const isEqual = require('lodash.isequal');
const moment = require('moment');
const diff = require('jest-diff');

function formatCurrency(num) {
    return (
        '$' +
        num.toString().replace(/(\d\d\d(?=[^.]))/g, function($1) {
            return $1 + ',';
        })
    );
}

function pickAndFormatTransaction({ amount, date, description }) {
    return {
        amount: Number(amount) ? formatCurrency(amount) : amount,
        date: moment(date).format('DD/MM/YYY'),
        description
    };
}
%}
const expected = {
    amount: '$0.00',
    date: '19/05/2019',
    description: 'yada yada yada',
};

const actual = pickAndFormatTransaction({
    amount: 0,
    date: 1558307309712,
    description: 'yada yada yada'
});

// MDN.io/console/assert
console.assert(
	isEqual(expected, actual),
	'EXPAND FOR PRETTY DIFF: \n%s', // Using string substitutions MDN.io/console#Usage
	diff(expected, actual)
); 

{% endrunkit %}
```
<span></span><figcaption>Run and Take Cover, This Will Fail!</figcaption>

Even if you know why `amount` is wrong, how would you go about adding `console.log` to the troubled code?

```
{% runkit
 
const isEqual = require('lodash.isequal');
const moment = require('moment');
const diff = require('jest-diff');

function formatCurrency(num) {
    return (
        '$' +
        num.toString().replace(/(\d\d\d(?=[^.]))/g, function($1) {
            return $1 + ',';
        })
    );
}
%}
const pickAndFormatTransaction = ( {
        amount,
        date,
        description,
} ) => ( {
        amount: Number( amount )
            ? formatCurrency( amount ) 
            : amount,
        date: moment( date ).format( 'DD/MM/YYY' ),
        description
} );

const expected = {
    amount: '$0.00',
    date: '19/05/2019',
    description: 'yada yada yada',
};

const actual = pickAndFormatTransaction({
    amount: 0,
    date: 1558307309712,
    description: 'yada yada yada'
});

// MDN.io/console/assert
console.assert(
	isEqual(expected, actual),
	'EXPAND FOR PRETTY DIFF: \n%s', // Using string substitutions MDN.io/console#Usage
	diff(expected, actual)
); 

{% endrunkit %}
```
<span></span><figcaption>🛠 Break open the code and try adding `console.log` to see why the amount isn't coming out right.</figcaption> 

I took a swing at it as well.
```js
const pickAndFormatTransaction = ( {
        amount,
        date,
        description,
} ) => {
	const formattedAmount = Number( amount )
            ? formatCurrency( amount ) 
            : amount
	console.log( Number( amount ), formattedAmount )
	return {
        amount: formattedAmount,
        date: moment( date ).format( 'DD/MM/YYY' ),
        description
} };
```

If yours is anything like mine, it's none too pretty. I suppose "pretty" is subjective but work with me here, it'll be worth it. 
`pickAndFormatTransaction`, is trying really hard to squeeze every feature out of the latest and greatest from JavaScript. Here's something a little more established. 

### Array.Chaining

```
{% runkit
 
function parseNumbers(num) {
    return Number(num) || 0;
}
function removeEvens(num) {
    return num % 2;
}
%}
const result = ['1', '2', 'zero' , 3, 4, 5]
    .map(parseNumbers)
    .filter(removeEvens)
    .reduce(( acc, v ) => Math.max(acc, v));
    
console.log(result);
{% endrunkit %}
```
<span></span><figcaption>❓There is no bug here ( at least I don't think there is ) but where would you put the `console.log` if there was?</figcaption>

Chaining a bunch of array functions together. `map`, `reduce`, and `filter` were some of the first indications of where ES6 and modern JS were heading. They are wonderfully succinct alternatives to the `for` and `while` loops. But they have the same issue as `pickAndFormatTransaction`. There's no room to fit a `console.log`. Again, a pinprick of a problem, but it's enough to ruin a good train of thought. And that's because `console.log` is one of the last things in JavaScript that _hasn’t_ been modernized. It's a remnant of an older JavaScript where everything was done in blocks like the `if` statement rather than in line like the ternary operator.

## `console.log -> undefined`
You may have noticed that when you’ve run the RunKit examples above you see the `console.log` output and then `undefined`. That `undefined` is the result of `console.log(...)`. This is why debugging is cumbersome as we have to bend over backward and around `console.log` or rather `undefined`. I have yet to see a time when I need that `undefined`. We could save ourselves plenty of work if `console.log` returned the value it was logging instead of `undefined`.

## `console.tap`

I've created `console.tap` to be the logging function modern JS has been missing.


```
{% runkit %}
console.tap = v => {
    console.log( v )
    return v
};
{% endrunkit %}
```
<span></span><figcaption><strong>Oo Aah</strong></figcaption><figcaption><sub>…wait, that's it?</sub></figcaption>

First, yes I'm adding it to the global console object. That is my choice, I'm a mad man. Second, the function revolves around simplicity. It takes one value, logs _that_ value, and returns _that_ value. To the calling function, and the context around it, nothing happens. Which means there is no extra overhead or setup to debugging.


### Comparisons 


#### Function Composition

```js
const user = storage.getItem( 'user' );
console.log( user )

const userID = getUserId(
	JSON.parse( user )
);
```

```
{% runkit
 
function getUserId( user ) {
	return user.id
}

const storage = {
    store: { user: '{"id":1}' },
    getItem( name ) {
        return this.store[name]
    }
}

%}

const userID = getUserId(
    JSON.parse(
        console.tap( storage.getItem( 'user' ) )
    )
);
{% endrunkit %}
```
<span></span><figcaption> 🛠 Move around `console.tap`. What do you get from `console.tap(JSON).parse`? </figcaption>

#### Chaining

```js
const filtered = arr
        .map( parseNumbers )
        .filter( removeOdds );
console.log( filtered );

const res = filtered.reduce( average );
```

```
{% runkit %}
const result = console.tap(arr
	.map(parseNumbers)
	.filter(removeOdds)) // <- this parens closes `tap`
	.reduce(average);
{% endrunkit %}
```
<span></span><figcaption>🛠 move around <strong>just</strong> the closing `)` for `console.tap` to see each result</figcaption>

`console.tap`  could have also been used on each part of the chain since each function produces an array.

And as for`pickAndFormatTransaction`, _that_ overachiever,  why don’t you give `tap` a try. 

#### The Ktchn Snk

```
{% runkit
 
const isEqual = require('lodash.isequal');
const moment = require('moment');

function formatCurrency(num) {
    return (
        '$' +
        num.toString().replace(/(\d\d\d(?=[^.]))/g, function($1) {
            return $1 + ',';
        })
    );
}
const transactionData = {
    amount: 100,
    date: Date.now(),
    description: 'Shrimp! Heaven! Now!',
    type: '',
    from: 'Daniel',
    to: 'Mom'
};
%}
const pickAndFormatTransaction = ( {
        amount,
        date,
        description,
} ) => ( {
        amount: Number( amount )
            ? formatCurrency( amount ) 
            : amount,
        date: moment( date ).format( 'DD/MM/YYY' ),
        description
} );

console.log(pickAndFormatTransaction(transactionData));
{% endrunkit %}
```
<span></span><figcaption>🛠 throw in a few `console.tap`'s and see what you can see.</figcaption>

Without adding anything but  `console.tap`  you can log the whole returning object, or to anything involved in amount and moment. For `description` you will still have to expand the shorthand to  `description: description`. 

## `Have Fun!`

I have created a module for `console.tap` that takes care of some extra details but the function declaration is so small you can write it yourself when you need it.

```
{% runkit %}
console.tap = v => {
    console.log(v);
    return v;
};

// or, for the bold
console.tap = v => (console.log(v), v);
{% endrunkit %}
```

[![NPM](https://nodei.co/npm/console.tap.png)](https://nodei.co/npm/console.tap/)
