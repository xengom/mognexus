---
layout: post
title:  IIS에서 "~"를포함한 정적파일을 찾을 수 없는 문제
date:   2025-08-13
last_modified_at: 2025-08-13
category: [DevOps]
tags: [Javascript, Vue.js, IIS, Webpack, Babel]
---


# Issue
현재 IIS를 통해 frontend(vue.js)를 Serving하고 있으나, vue.js의 bundling된 결과물에 특수문자 "~"가 포함되는 경우 해당 파일을 불러올 수 없는 Issue가 있었다.<br/>
예를 들어 파일명이 `AAA.0000.js`인 경우 정상적으로 GET 요청을 통해 호출 가능하지만, 파일명이 `AAA~AAZ.0000.js`인 경우 아무리 호출하더라도 500 error를 Return하는 식이다. 

<br/><br/>

# Cause by
IIS 상의 web.config, applicationHost.config에 별다른 설정이 없음에도 불구하고 특수문자 "~"가 Filtering되고 있어서 확인해본 결과,
3rd Party 프로그램인 siteminder를 web.config에서 사용중이고 해당 프로그램의 ISAPI Filter 설정에서 특수문자를 차단하고 500 error를 Return 하고 있었다. 
실제로는 404 error여야 하겠지만, 보안상 모든 에러를 500 error로 치환해서 내보내고 있는 것으로 보인다.

<br/><br/>

# Solution
IIS 설정으로 특수문자 "~"를 허용하는 것은 리스크가 크기 때문에 (IIS에서 "~"는 root 경로를 의미) <br/> 가장 이상적인 해결방안은 vue.js를 Build할 때, Bundling되는 결과물의 파일명 중, 특수문자 "~"를 IIS상에서 "-"와 같은 허용된 다른 문자로 치환하면 된다. (ex. `AAA~AAZ.0000.js` -> `AAA-AAZ.0000.js`)

<br/>

```js
// vue.config.js
chainWebpack: config => {
    config.output.chunkFilename('js/[name].js');
    config.optimization.splitChunks({
        name(module, chunks) {
            return chunks.map(chunk => chunk.name).join('-');
        }
    });
    // ... 기존 Webpack 설정...
}
```


<br/><br/>
