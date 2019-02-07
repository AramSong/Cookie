## HTTP

Hypertext Transfer Protocol

통신 방법.

웹브라우저와 웹서버가 데이터를 주고받을 때,  

>![1549506786637](https://user-images.githubusercontent.com/38032500/52395937-39ffdd80-2af3-11e9-81bf-792f35b533d9.png)
 [출처] 생활코딩 Server Side js-http



![1549506890949](https://user-images.githubusercontent.com/38032500/52395938-39ffdd80-2af3-11e9-8d0d-361e4bba1466.png)
![1549506933607](https://user-images.githubusercontent.com/38032500/52395939-39ffdd80-2af3-11e9-8d0b-10b2f8b65ea3.png)


1.`Response Headers`에 요청한 내용이 나와 있음. 웹서버는 response header를 통해 웹브라우저에 응답함.

2.웹브라우저는 이 내용을 참고해서 작업들을 처리.

3.User-Agent는 어떤 웹브라우저를 참조하고 있는지 나와있음.

> "웹브라우저와 웹서버는 http라는 통신방식을 통해서 서로가 정보를 주고받는다. 통신방법의 핵심은 `Request Header`와 `Response Header` 를 이용하여 데이터들에 정보를 확인할 수 있다."

### Cookie 1

 접속할 때마다 서버쪽에서 그 내용을 알고 있음.(로그인 유지)

이전에 접속한 상태를 다음 상태가 알 수 있음.

서로 다른 창에서는 내용을 공유하지 않음.(사용자마다 다른 상태를 유지)

* cookie-parser 미들웨어 설치

![1549519938531](https://user-images.githubusercontent.com/38032500/52395940-39ffdd80-2af3-11e9-8f0d-bbb480dee2f9.png)

![1549520160449](https://user-images.githubusercontent.com/38032500/52395941-3a987400-2af3-11e9-8300-9aebba2f1dc9.png)


* 웹브라우저가 웹서버에게 count=1이라는 쿠키를 전송.
* 새로고침을 할 때마다 count가 1씩 증가하는 프로그램을 구성.

`app_cookie.js`

```javascript
var express = require('express');
var cookieParser = require('cookie-parser');
var app = express();
app.use(cookieParser());
app.get('/count',function(req,res){
//cookie값이 있을 경우,
  if(req.cookies.count){
    var count = parseInt(req.cookies.count);
  }
 //cookie값이 없을 경우
  else{
    var count = 0;
  }
  count = count + 1;
  res.cookie('count',count);
  res.send('count : ' + req.cookies.count);
});
app.listen(3003,function(){
  console.log('Connected 3003 port!!!');
});

```

### Cookie2
![1549520880224](https://user-images.githubusercontent.com/38032500/52395942-3a987400-2af3-11e9-9113-7c3b8170929d.png)

![1549520892257](https://user-images.githubusercontent.com/38032500/52395943-3a987400-2af3-11e9-9d70-46fbd2d51b8d.png)


반복문이 실행되면서 배열의 내용을 하나씩 출력함.

1. Shopping Cart 1 : 상품을 클릭하면 cart1, cart2으로 가게 함

```javascript
app.get('/products',function(req,res){
  var output = '';
  for(var name in products){
    output += `
    <li>
      <a href="/cart/${name}">${products[name].title}</a>
    </li>`
  }
  res.send(`<h1>Products</h1>
            <ul>${output}</ul>
            <a href="/cart">Cart</a>`);
});
```

2. Shopping Cart 2 

```javascript
/*
cart = {
//제품의 id : 제품의 수량(cookie증가)
  1:2,
  2:1,
}
*/
app.get('/cart/:id',function(req,res){
  var id = req.params.id;
  if(req.cookies.cart){
    var cart = req.cookies.cart;
  }else{
    var cart = {};
  }
  if(!cart[id]){
    cart[id] = 0;
  }
  cart[id] = parseInt(cart[id]) + 1;
  res.cookie('cart',cart);
  res.redirect('/cart');
});
```

3. Shopping Cart 3

```javascript
pp.get('/cart',function(req,res){
  var cart = req.cookies.cart;
  if(!cart){
    res.send('Empty!');
  }else{
    var output = '';

    for(var id in cart){
      //제품의 id값을 출력
      //output += `<li>${id}</li>`;
      //해당 id의 상품의 제목을 출력. cart[id] :해당 id의 수량을 나타냄
      output += `<li>${products[id].title}
       : ${cart[id]}</li>`
    }
  }
  res.send(`
  <h1>Cart</h1>
  <ul>${output}</ul>
  <a href ="/products">Products List</a>`);
});
```

4. Cookie & Security :  cookie를 서버와 웹브라우저가 주고받는 상황에서 누군가가 중간에서 쿠키값을 볼 수 있음. 로그인과 관련된 쿠키같은 중요한 정보일 경우 심각한 보안상의 구멍이 되므로 이에 대한 노출을 막기 위함.
![1549522635142](https://user-images.githubusercontent.com/38032500/52395944-3b310a80-2af3-11e9-9981-86fcfdd74ef8.png)

괄호안의 내용이 key값이 됨. 서버에서 브라우저로 cookie를 구울 때, 암호화해서 cookie를 구움. 서버에 Request할 때 암호화된 정보를 보내서, key값을 통해 암호화된 정보를 알아냄.

![1549522721685](https://user-images.githubusercontent.com/38032500/52395945-3b310a80-2af3-11e9-8bfe-26849408630e.png)
![1549522921421](https://user-images.githubusercontent.com/38032500/52395946-3b310a80-2af3-11e9-8b04-639f2a911250.png)

사용자의  COOKIE값이 signdedCookies를 통해서 key를 통해 암호화가 풀린 값으로 전달됨.

```javascript
  if(req.signedCookies.cart){
    var cart = req.signedCookies.cart;
  }else{
    var cart = {};
  }
  if(!cart[id]){
    cart[id] = 0;
  }
  cart[id] = parseInt(cart[id]) + 1;
  res.cookie('cart',cart,{signed:true});
```

