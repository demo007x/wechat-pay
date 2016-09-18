#wechatPay
# nodejs 微信公众号支付开发

---

NodeJs 微信公众号功能开发,移动端 H5页面调用微信的支付功能。这几天根据公司的需要使用   node 和 h5页面调用微信的支付功能完成支付需求。现在把开发过程重新捋一遍，以帮助更多的开发者顺利的完成微信支付功能的开发。（微信暂时还没有提供 node 的支付功能）


----------

#### 一.请求CODE

    请求 code 的目的就是获取用户的 openid（用户相对于当前公众号的唯一标识） 和access_token，请求的 **API**：https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect
此 api 需要注意几个参数：

    1.  appid公众号的 appid，可以在公众号中看到
    2.  redirect_uri 自定义的微信回调地址， 微信会在你请求完上面的地址后跳转到你定义的redirect_uri的地址， 带着 code，此处的 redirect_url 需要 **url_encode** *php*， 如果你的程序是 node 则需要使用 **encodeURLComponent(url)**编码
    3.  response_type=code，这个没什么好说的就是固定的 response_type=code，详细说明可以查看微信官网的说明
    4.  scope=snsapi_userinfo，固定这样写就好，详细说明可以查看微信官网的说明
    5.  state=STATE 固定这样写就好，详细说明可以查看微信官网的说明
    6.  wechat_redirect 固定这样写就好，详细说明可以查看微信官网的说明
   
ps：官网链接：


----------
#### 二.通过code获取access_token,openid
第一步已经获取到了 code 的值了， 那么接下来就需要通过 code 来获取 access_token,openid的值了，请求的 api
**API** https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
此处api 的参数说明：

    1.  appid 微信公众号 id，微信公众号后台获取
    2.  secret 微信公众号的密钥， 微信公众号后台获取
    3.  code， 第一步获取用到的 code
    4.  grant_type=authorization_code 固定就好
    
--------
    
###### 三.通过access_token调用接口
access_token 可以做后续的功能， 可以参考官方的例子：
[https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419316518&lang=zh_CN][1]

--------

#### 四.网页端调起支付API
看见这个是不是感觉快完事儿了， 只要网页端调用微信支付功能就完事儿了？no，还差点
在微信浏览器里面打开H5网页中执行JS调起支付。接口输入输出数据格式为JSON。
注意：WeixinJSBridge内置对象在其他浏览器中无效。
**示例代码如下：**

    function onBridgeReady(){
       WeixinJSBridge.invoke(
           'getBrandWCPayRequest', {
               "appId" ： "wx2421b1c4370ec43b",     //公众号名称，由商户传入     
               "timeStamp"：" 1395712654",         //时间戳，自1970年以来的秒数     
               "nonceStr" ： "e61463f8efa94090b1f366cccfbbb444", //随机串     
               "package" ： "prepay_id=u802345jgfjsdfgsdg888",     
               "signType" ： "MD5",         //微信签名方式：     
               "paySign" ： "70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名 
           },
           function(res){     
               if(res.err_msg == "get_brand_wcpay_request：ok" ) {}     // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。 
           }
       ); 
    }
    if (typeof WeixinJSBridge == "undefined"){
       if( document.addEventListener ){
           document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
       }else if (document.attachEvent){
           document.attachEvent('WeixinJSBridgeReady', onBridgeReady); 
           document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
       }
    }else{
       onBridgeReady();
    }
    
看到上面的代码， 那么想调用微信的支付功能需要传递参数，

    {
       "appId" ： "wx2421b1c4370ec43b",     //公众号名称，由商户传入     
       "timeStamp"：" 1395712654",         //时间戳，自1970年以来的秒数     
       "nonceStr" ： "e61463f8efa94090b1f366cccfbbb444", //随机串     
       "package" ： "prepay_id=u802345jgfjsdfgsdg888",     
       "signType" ： "MD5",         //微信签名方式：     
       "paySign" ： "70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名 
    }
    
参数说明：

    1. appId //公众号名称，由商户传入 
    2. timeStamp //时间戳，自1970年以来的秒数  此处需要特别的注意一下，需要是字符串的时间戳格式， 意思就是必须就“” 引号
    3. nonceStr //随机串    32位的， 随后会提供方法
    4. signType // 微信签名方式： MD5
    5. paySign //微信签名， 随后说
    6. **package**   //这个最重要， 充哪里获取到的呢？ 接下来说。
    
ps: 官网接口说明
[https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6][2]

--------
#### 五.获取package， 从微信的 统一下单 接口获取prepay_id
官方 api： 
[https://api.mch.weixin.qq.com/pay/unifiedorder][3]


 请求参数一堆， 但是有一些不是必须的，下面是必须参数
 

    {
        appid : APPID,
        attach : ATTACH,
        body : BODY,
        mch_id : MCH_ID,
        nonce_str: NONCE_STR,
        notify_url : NOTIFY_URL,// 微信付款后的回调地址
        openid : OPENID,
        out_trade_no : OUT_TRADE_NO ,//new Date().getTime(), //订单号
        spbill_create_ip : SPBILL_CREATE_IP , //客户端的 ip
        total_fee : TOTAL_FEE, //商品的价格， 此处需要注意的是这个价格是以分算的， 那么一般是元， 你需要转换为 RMB 的元
        trade_type : 'JSAPI',
    }
    
    
微信的统一下单接口要求传递的是 xml 的数据， 而且数据还需要签名， 那么首先吧数据签名。
签名规则可以参考微信给出的签名规则（签名方法一会给出）
微信官方签名规则：
[https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=4_3][4]


  [1]: https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419316518&lang=zh_CN
  [2]: https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6%20%E5%AE%98%E7%BD%91%E8%AF%B4%E6%98%8E
  [3]: https://api.mch.weixin.qq.com/pay/unifiedorder%20%E5%AE%98%E6%96%B9%E7%BB%9F%E4%B8%80%E4%B8%8B%E5%8D%95API
  [4]: https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=4_3%E5%BE%AE%E4%BF%A1%E5%AE%98%E6%96%B9%E7%AD%BE%E5%90%8D%E8%A7%84%E5%88%99
  
  
生成签名后需要吧数据组装为xml 的格式：

    var body = '<xml> ' +
        '<appid>'+config.wxappid+'</appid> ' +
        '<attach>'+obj.attach+'</attach> ' +
        '<body>'+obj.body+'</body> ' +
        '<mch_id>'+config.mch_id+'</mch_id> ' +
        '<nonce_str>'+obj.nonce_str+'</nonce_str> ' +
        '<notify_url>'+obj.notify_url+'</notify_url>' +
        '<openid>'+obj.openid+'</openid> ' +
        '<out_trade_no>'+obj.out_trade_no+'</out_trade_no>'+
        '<spbill_create_ip>'+obj.spbill_create_ip+'</spbill_create_ip> ' +
        '<total_fee>'+obj.total_fee+'</total_fee> ' +
        '<trade_type>'+obj.trade_type+'</trade_type> ' +
        '<sign>'+obj.sign+'</sign> ' + // 此处必带签名， 否者微信在验证数据的时候是不通过的
        '</xml>';
        
接下来就是请求 api 获取prepay_id的值了， 将上面得到的 xml 数据请求下面的 api 发送给微信， 微信验证数据没问题后会放回你想要的值。
api ： https://api.mch.weixin.qq.com/pay/unifiedorder

--------

#### 六. 获取到了prepay_id是不是就可以在 h5 段直接调用微信的支付了么？ 答案是还不可以。
获取到了prepay_id那么现在h5 呼起微信的支付功能的参数是这样的：

    {
       "appId" ： "wx2421b1c4370ec43b",     //公众号名称，由商户传入     
       "timeStamp"：" 1395712654",         //时间戳，自1970年以来的秒数     
       "nonceStr" ： "e61463f8efa94090b1f366cccfbbb444", //随机串     
       "package" ： "prepay_id=u802345jgfjsdfgsdg888",     
       "signType" ： "MD5",         //微信签名方式：
    }
  
有了这样的参数， 那么你还需要吧所有参与的参数做签名。签名规跟上面的一样，生成了签名后需要吧签名的参数  paySign 赋给h5 呼起微信的支付功能的参数（也就是微信的签名不参与签名的生成）
最后的参数是这样子的：

    {
       "appId" ： "wx2421b1c4370ec43b",     //公众号名称，由商户传入     
       "timeStamp"：" 1395712654",         //时间戳，自1970年以来的秒数     
       "nonceStr" ： "e61463f8efa94090b1f366cccfbbb444", //随机串     
       "package" ： "prepay_id=u802345jgfjsdfgsdg888",     
       "signType" ： "MD5",         //微信签名方式：
       "paySign" ： "70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名
    }
    
如果你的各个环节都没有问题， 那么得到了这样参数后你就可以正常的调用起微信的支付功能， 跟原生的功能是没有任何的差别的，（估计你现在的心里也特高兴吧， 没有 app 竟然可以使用 app 的功能，就是这么的神奇）。

--------

#### 7.支付完成的回调
微信支付完了后会在 h5 页面的微信支付的回调函数里面放回值，
res.err_msg == "get_brand_wcpay_request：ok" ，这样就是成功了， 但是不是就完事儿了呢 ？ 也不是，为什么呢？ 微信真的收到钱了么？ 收到的钱是不是你传递给微信的值呢 ？你还需要将支付的结果写数据库什么的，这些都是未知。还记的在统一下单接口中有个必须参数就是 `notify_url : NOTIFY_URL,// 微信付款后的回调地址` 这个地址是用户传递给微信的， 微信在收到用户的付款后会以 **post** 的方式请求这个接口，微信会传递用户付款的信息过来， 不过是 xml 格式的。
类系这样的 xml 格式：

    <xml>
      <appid><![CDATA[wx2421b1c4370ec43b]]></appid>
      <attach><![CDATA[支付测试]]></attach>
      <bank_type><![CDATA[CFT]]></bank_type>
      <fee_type><![CDATA[CNY]]></fee_type>
      <is_subscribe><![CDATA[Y]]></is_subscribe>
      <mch_id><![CDATA[10000100]]></mch_id>
      <nonce_str><![CDATA[5d2b6c2a8db53831f7eda20af46e531c]]></nonce_str>
      <openid><![CDATA[oUpF8uMEb4qRXf22hE3X68TekukE]]></openid>
      <out_trade_no><![CDATA[1409811653]]></out_trade_no>
      <result_code><![CDATA[SUCCESS]]></result_code>
      <return_code><![CDATA[SUCCESS]]></return_code>
      <sign><![CDATA[B552ED6B279343CB493C5DD0D78AB241]]></sign>
      <sub_mch_id><![CDATA[10000100]]></sub_mch_id>
      <time_end><![CDATA[20140903131540]]></time_end>
      <total_fee>1</total_fee>
      <trade_type><![CDATA[JSAPI]]></trade_type>
      <transaction_id><![CDATA[1004400740201409030005092168]]></transaction_id>
    </xml>
    
根据自己的业务逻辑解析这个 xml 格式的数据就好了。
注意：这里你在获取到数据后微信需要得到你的回应， 如果你一直不回应微信， 微信会请求你好几次， 这样估计你的逻辑会有问题吧，所以你需要给微信返回 xml 的格式的 回应。

    <xml>
      <return_code><![CDATA[SUCCESS]]></return_code>
      <return_msg><![CDATA[OK]]></return_msg>
    </xml>
    
--------

小坑：node ，express 框架开发， 如果你在微信的支付成功后的回调中没有获取到任何 xml 的值， 那么你需要安装一组件：body-parser-xml， 你可以使用 `npm install body-parser-xml --save` 安装， 在 app.js 里面 `require('body-parser-xml')(bodyParser);`,使用中间件的方式

    // 解决微信支付通知回调数据
    app.use(bodyParser.xml({
      limit: '2MB',   // Reject payload bigger than 1 MB
      xmlParseOptions: {
        normalize: true,     // Trim whitespace inside text nodes
        normalizeTags: true, // Transform tags to lowercase
        explicitArray: false // Only put nodes in array if >1
      }
    }));
    
这样你就可以正常的获取到微信的 xml 数据了。

--------
#### 使用方法：
    
    `pay.getAccessToken({
            notify_url : 'http://demo.com/', //微信支付完成后的回调
            out_trade_no : new Date().getTime(), //订单号
            attach : '名称',
            body : '购买信息',
            total_fee : '1', // 此处的额度为分
            spbill_create_ip : req.connection.remoteAddress,
        }, function (error, responseData) {
            res.render('payment', {
                title : '微信支付',
                wxPayParams : JSON.stringify(responseData),
                //userInfo : userInfo
            });
        });`
                
#### 就到这里吧， 感觉也差不多了。如有不对的地方还请指正。
    

    
        
