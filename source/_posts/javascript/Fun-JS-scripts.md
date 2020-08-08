---
title: 推荐几个实用或好玩的js脚本
date: 2020-09-23 20:55:00
tags: javascript
categories: javascript
---
简单的几个脚本,不像TamperMonkey那样强植入, 也无需去Chrome应用商店安装什么扩展插件.
<!--more-->

**如果未来有新脚本, 会陆续在这里更新.**

1. 解除网页禁止复制的限制（同时去除复制内容末尾自动附加的额外小尾巴信息）
```javascript
  javascript:window.oncontextmenu=document.oncontextmenu=document.oncopy=null; [...document.querySelectorAll('body')].forEach(dom => dom.outerHTML = dom.outerHTML); [...document.querySelectorAll('body, body *')].forEach(dom => {['onselect', 'onselectstart', 'onselectend', 'ondragstart', 'ondragend', 'oncontextmenu', 'oncopy'].forEach(ev => dom.removeAttribute(ev)); dom.style['user-select']='auto';});
```
2. 密码输入框明文显示（方便查看被浏览器记住但自己已经忘了的密码）
```javascript
  javascript:[...document.querySelectorAll('input[type=password]')].forEach(i => i.type = 'text');
```
3. 给网页内所有元素添加随机颜色外框线
```javascript
  javascript:[...document.querySelectorAll('*')].forEach(i => { let rand = (~~(Math.random() * 0xFFFFFF)).toString(16); rand = '#' + ('00000' + rand).slice(-6); i.style.outline = '1px solid ' + rand; });
```
4. 点击页面随机出现向上浮动并淡出的小字
```javascript
  javascript:(()=>{if(document.querySelector('#clickAnimation_')) return; var rand=()=>{var arr=['富强', '民主', '文明', '和谐', '自由', '平等', '公正', '法治', '爱国', '敬业', '诚信', '友善']; return arr[~~(Math.random()*arr.length)];}; var s=document.createElement('style'); s.id='clickAnimation_'; s.innerText='.click-elem-t {position: absolute; z-index: 9999; color: #f5871f; font-size: 15px; font-weight: bold; cursor: default; user-select: none; animation: floatAndGone 1.7s forwards;} @keyframes floatAndGone {from {transform: translateY(0); opacity: 1;} to {transform: translateY(-75px); opacity: 0;}}'; document.querySelector('head').appendChild(s); document.body.addEventListener('click', e=>{var dom=document.createElement('span'); dom.innerText=rand(); dom.className='click-elem-t'; dom.style.top=e.pageY-25+'px'; dom.style.left=e.pageX+'px'; document.body.appendChild(dom); setTimeout(()=>{document.body.removeChild(dom)}, 2200) });})();
```
5. 解除所有按钮的禁用状态
```javascript
  javascript:[...document.querySelectorAll('input[type=button], button')].forEach(i => i.removeAttribute('disabled'));
```
6. 解除所有文本输入框/单选框/复选框的只读和禁用状态
```javascript
  javascript:[...document.querySelectorAll('textarea, input')].forEach(i => { i.removeAttribute('disabled'); i.removeAttribute('readonly') });
```


