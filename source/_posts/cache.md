---
title: 缓存与页面自动更新
tags: web git 
categories: web
abbrlink: 1
date: 2018-10-05 00:40:49
---
### 遇到的问题：
当浏览器访问某个文件（比如图片、css文件、js文件）时，若该文件有缓存且尚未失效，那么浏览器会直接从缓存中读取该文件，即使服务器上该文件已经发生了更新。然而有些情况下，我们需要客户在页面使用该文件的最新版本，即便它在浏览器被缓存。

解决思路：
1、每次文件在服务器有更新，上线之后告诉所有用户，要清除浏览器缓存才可以使用；
优点：简单省事  缺点：给运营及客户造成不便，容易挨揍
2、强制文件不使用缓存，每次浏览器访问都需要从服务器获取；
优点：开发成本低  缺点：页面访问速度下降；服务器压力增加
3、更聪明的办法
<!-- more -->

### 我们的目标：
   当文件在服务器没有更新时，保持原来的缓存机制；当文件在服务器有更新时，从服务器获取最新版本。

### 背景：
先看一下浏览器缓存的分类，
1、强缓存：由expires和cache-control控制，当响应头中包含这两个header时，浏览器再次请求同样的资源，不会和服务器发生交互，而是直接从本地缓存中获取，除非缓存过期。

2、协商缓存：由Etag和Last-Modified控制，当响应头中包含这两个header时，浏览器每次获取同样的资源，会向服务器发请求，但是服务器未必会返回资源本身，而是可能返回一个304响应,告诉浏览器该资源没有发生变化，可以直接使用本地的缓存。Last-Mofified记录的是文件的上次修改时间，服务器会比较请求（请求的header是If-Modified-Since，但值一样）中文件上次修改时间的值与文件真实的修改时间，当二者不一致时，服务器才会在响应体中写入最新的文件内容。

 似乎Last-Modified或Etag这两个协商缓存的header就可以解决这个问题。但是这两个header一般是和强缓存的header一起工作的，也就是一个相应中会同时存在这四个header，这时缓存是这样工作的：

 即使让协商缓存的header单独工作，那么不可避免的，每次请求同样的资源，浏览器还是会频繁的向服务器发请求，对服务器仍会造成很大的压力。

### 聪明的方案：
一句话总结：更改资源的url。
对于同一份（内容已改，只是文件名不变）文件，每次更新后，更改访问它的url，那么浏览器会将其视为一个新文件，从而是第一次访问，自然不会使用缓存。比如在index.html中引用了xx.css：
```
<link rel="stylesheet" type="text/css" href=“xx.css”/>
```
当xx.css的内容更新后，需要同时改变index.html的link标签href属性的值，也就是访问xx.css的url。但是无论怎么改，还需要保证服务器能够将这个url解析到xx.css上。

### url怎么改？
方案1：
将xx.css改为xx_<version>.css
方案2：
将xx.css改为xx.css？v=<version>
比较：
   两种方案都是可行的，方案1直接更改文件名，实现上更为复杂一点，但是更健壮；方案2通过添加query string的方式将url改写，实现更简单，但是有些问题，后面会说。
   wifi链采用了方案2,服务器使用了springboot框架，将特定路径的url视为静态资源访问，会直接忽略掉url后面的query string，所以能够将不同的url解析到同名文件上。
    另外，version的生成实际上采用了一个插件git-commit-id-plugin，可以获取到git commitId，方法是在springboot的config类中加入以下代码：
    

    @Value("${git.commit.id.abbrev}")
    private String buildVersion;
    @Bean
    InitParameterConfiguringServletContextInitializer 
        initParameterConfiguringServletContextInitializer() {
        Map<String, String> contextParams = new HashMap<>();
        contextParams.put("buildVersion", buildVersion);
        return new InitParameterConfiguringServletContextInitializer(contextParams);
    }

然后在jsp中引用这个context中attribute，动态生成html文件：
```
<link href=app.css?v=${initParam.buildVersion} rel=stylesheet>
```
### 方案2的缺陷：
   方案2每次更新都会涉及到至少两个文件，一个html文件和一个缓存文件，如上例中的index.html和xx.css。
    现在的大型web应用往往采用前后端分离的方式部署，前端的静态资源会单独部署在CDN节点服务器上，而html页面往往是在应用服务器上动态生成。 所以上线时，静态资源和页面的部署总会有一个时间差，无论这个时间差有多小，对于访问量大的网站，都会造成页面和静态资源的不一致从而导致页面错误。
    解决这个问题的方法就是方案1：静态资源更新后名字会发生改变，可以先以新文件的姿态部署到CDN，不会覆盖原来的旧文件；然后再部署页面，就可以实现平滑升级。
    另外，方案2还有一个缺陷，如果一个页面中引用了多个静态资源，但是某次上线只有一个静态资源更新了，那么其他所有的静态资源的版本号都会发生改变，从而浏览器需要重新从服务器拉取这些文件，即使它们没有更新，这导致了浪费。
