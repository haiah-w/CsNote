# CSRF跨站请求伪造

场景：  
1、用户A，登录网站B，没有登出；  
2、用户A，同时访问了网站C（攻击者）；  
3、网站C返回的信息中，存在攻击js代码，自动执行了网站B的接口，并且携带了浏览器中网站B的Cookie；  
4、如果网站B没有防范CSRF，就会认为接口执行者是用户本人；  

# Http Referer

所有的站内操作，都应该验证Http Referer字段，是否是自己本站点发起的，不接受外部站点的请求，直接操作重要接口；  Http Referer：指明发起请求的域名； 
特点：  
1、简单易行  
2、依赖与浏览器的安全程度，因为Http Referer的值是浏览器提供的；  

# 隐藏域Token

用户登陆认证完成，服务端生成Token  
token由前端保存，每次用户从前端发起请求，会设置域token，再由后端验证；  
token不会公开，防止攻击者获取；  

# 验证码

特殊场景可以使用验证码，但是并不能覆盖很全，影响用户体验；  
目的就是验证操作者就是用户本人；
