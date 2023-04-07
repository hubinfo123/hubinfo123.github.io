---
title: download 下载方式
date: 2020-10-29 19:15
tags: [JS]
categories: js 基础
excerpt: window.open 在cookie 与token 鉴权下有不同的表现形式。
---

<font face="STCAIYUN" size="3">今天在于后端调试一个下载接口是，遇到一个下载不成功，提示’无权限‘操作，绕了一圈发现是此接口做了权限效验，草率了...</font>


![](/images/download.png)

> <font face="STCAIYUN" size="2">原因分析：
1.后端提供一个下载接口，接口鉴权走中间件middlewoare ,
  2.前端为了方便，使用window.open 直接拼装了链接，进行了下载，
  3.2个系统，同一个接口，同一份代码，在a 端就可以正常下载，在b 端就下载不了</font>

<font face="STCAIYUN" size="2">导致在2端出现不同的现象是，a 端鉴权使用Cookie ,b 端鉴权使用 token ,Cookie 情况下无论使用ajax ,还是 window.open 在header 里面默认就被添加，而token 这种就做不到，所以就会导致同样的代码，在不同的鉴权下就现象不同。</font>


```
解决方法：
const downloadOriginalText = useCallback(() => {
    // 接口
    exportDownloadApi({ ct_id:ctId, task_id:taskId, ca_id:caId }).then(({ data }) => {
      const url = window.URL.createObjectURL(data);
      let a = document.createElement('a');
      document.body.appendChild(a);
      // 可以将 a 标签display 设为none ,以免有所影响
      a.href = url;
      a.click();
      window.URL.revokeObjectURL(url);
    })
  }, [ctId, taskId, caId])
```

> <font face="STCAIYUN" size="2">上述方法可能会出现的问题：
1.设置responseType ,拿使用axios exportDownloadApi 这个接口的地方要设置responseType ,它是跟Header 同级</font>
```
export async function exportDownloadApi (data) {
  return requestHelper.get('/exportxxxxxx00000000', { params: data, responseType: 'blob' });
}
```
> <font face="STCAIYUN" size="2">2.在公共处理请求的地方，在header 里面设置token</font>

<font face="STCAIYUN" size="2">如果不设就会出现，点击下载，浏览器会出现一个下载行为，文件也下载下载下来了，就是无法打开，告知被损坏，正常下载是没有这个下载行为的，直接就打开了（有待验证）</font>


<font face="STCAIYUN" size="2">在调试过程中还发生这样一个问题：
问题：设置了responseType 也设置了token ,但是修改了Response Header 里面的Content-Type: application/octet-stream,下载下来的也是损坏的，下载动作会有。此时 Response Header 里面是：
Content-Disposition: attachment; filename="translation-55980.zip"
Content-Type: application/octet-stream，没有具体去分析，感觉问题是出现在这里，于是又让后端设置为：
Content-Type: application/zip，一把过。</font>



