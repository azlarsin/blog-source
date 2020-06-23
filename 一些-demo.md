title: 一些 demo
author: azlar
date: '2019-03-26 20:54:10'
tags: []

---

<!-- desc -->

## some demos
jsfiddle order: /result,js,html,css/

### debounce & throttle
<div style='margin: 27px 0;box-shadow: 0 0 7px 1px rgba(0,0,0, .15)'>
	<iframe width="100%" height="300" src="//jsfiddle.net/azlar/07ua649y/100/embedded/result,js,html,css/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
</div>


#### code
##### debounce
```javascript
// debounce
function debounce(fn, time = 100, options = {}) {
  let args = Array.prototype.slice.call(arguments);
  let timeout = null;
  let debouncing = false;

  let {
    leading
  } = options;
  
  return function() {

    if (leading) {
    	console.log('x', timeout);
      if (!timeout && !debouncing) {
        fn.apply(this, args);
        debouncing = true;

        setTimeout(() => {
          debouncing = false;
          leading = false;
          
          timeout = setTimeout(() => {
            leading = true;
            debouncing = false;
            timeout = null;
          }, time);
        }, time);
      }
    } else {
      	clearTimeout(timeout);
        timeout = setTimeout(() => {
          console.log('xxx', debouncing);
          debouncing = false;
          timeout = null;
          fn.apply(this, args);
          
          leading = true;

        }, time);
    
    }
  }

}
```


##### throttle
```javascript
// throttle
function throttle(fn, freq = 33.33333) {
  let args = Array.prototype.slice.call(arguments);

  let processing = false;
  return function() {
    if (!processing) {
      processing = true;

      setTimeout(() => {
        fn.apply(this, args);
        processing = false;
      }, freq);
    }
  }
};
```


### infinite slider
<div style='margin: 27px 0;box-shadow: 0 0 7px 1px rgba(0,0,0, .15)'>
	<iframe width="100%" height="300" src="//jsfiddle.net/azlar/81pkteyc/193/embedded/result,js,html,css/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
	
</div>	

### background-attachment: fixed;
<div style='margin: 27px 0;box-shadow: 0 0 7px 1px rgba(0,0,0, .15)'>
	<iframe width="100%" height="300" src="//jsfiddle.net/azlar/38wpfyLd/2/embedded/result,js,html,css/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
</div>


### moving background(linear-gradient)
<div style='margin: 27px 0;box-shadow: 0 0 7px 1px rgba(0,0,0, .15)'>
	<iframe width="100%" height="300" src="//jsfiddle.net/azlar/koqc94eu/1/embedded/result,html,css/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
</div>


### IOS Loading (pull to request)
<div style='margin: 27px 0;box-shadow: 0 0 7px 1px rgba(0,0,0, .15)'>
	<iframe width="100%" height="640" src="//jsfiddle.net/azlar/dg0j8ton/10/embedded/result,html,css/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
</div>