```
._   _       _            
| \ | | ___ | |_ ___  ___
|  \| |/ _ \| __/ _ \/ __|
| |\  | (_) | ||  __/\__ \
|_| \_|\___/ \__\___||___/

```

# Performance

## Reduce bundle size
[cost-of-modules: Find out which of your dependencies are slowing you down 🐢](https://github.com/siddharthkp/cost-of-modules)

## How to analyze perf
#### The simplest way
```js
var t0 = performance.now();
doSomething();
var t1 = performance.now();
console.log("Call to doSomething took " + (t1 - t0) + " milliseconds.");
```

#### The better way
Use [marky](https://github.com/nolanlawson/marky). It works in node and legacy browsers.

```js
var marky = require('marky');

marky.mark('expensive operation');
doExpensiveOperation();
marky.stop('expensive operation');
```

## Written notes that'll be add here later
![](https://github.com/kylpo/web-playbook/blob/master/assets/perf.jpeg?raw=true)

![](https://github.com/kylpo/web-playbook/blob/master/assets/perf1.jpeg?raw=true)

![](https://github.com/kylpo/web-playbook/blob/master/assets/perf2.jpeg?raw=true)