#### Typora结合PicGo上传成功但不显示图片

![image-20220719222253780](https://raw.githubusercontent.com/kurisaW/picbed/main/img/202207192222855.png)

此处的设定自定义域名不需要填写，否则还需要自行设定



---

## csdn上传GitHub仓库图片不成功

解决方案：
打开host文件

![image-20220719222538146](C:/Users/ASUS/AppData/Roaming/Typora/typora-user-images/image-20220719222538146.png)

加入以下内容

```
# GitHub Start 
192.30.253.112    Build software better, together 
192.30.253.119    gist.github.com
151.101.184.133    assets-cdn.github.com
151.101.184.133    raw.githubusercontent.com
151.101.184.133    gist.githubusercontent.com
151.101.184.133    cloud.githubusercontent.com
151.101.184.133    camo.githubusercontent.com
151.101.184.133    avatars0.githubusercontent.com
151.101.184.133    avatars1.githubusercontent.com
151.101.184.133    avatars2.githubusercontent.com
151.101.184.133    avatars3.githubusercontent.com
151.101.184.133    avatars4.githubusercontent.com
151.101.184.133    avatars5.githubusercontent.com
151.101.184.133    avatars6.githubusercontent.com
151.101.184.133    avatars7.githubusercontent.com
151.101.184.133    avatars8.githubusercontent.com

 # GitHub End
```

