---
title: 推荐几个实用或好玩的js脚本
date: 2020-09-23 20:55:00
categories: 前端
tags: ['javascript','js脚本']
---
简单的几个脚本,不像TamperMonkey那样强植入, 也无需安装浏览器扩展插件.
<!--more-->
### 脚本列表
**如果未来有新脚本, 会陆续在这里更新.**
*具体使用方法在页面底部*
1. 解除网页禁止复制的限制（同时去除复制内容末尾自动附加的额外小尾巴信息
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

> 如果你想要其他脚本，在评论区回复吧.

### 使用方法
*以下三种方法可任选其一*
1. 直接复制脚本代码，然后粘贴浏览器的地址栏，然后回车确定（chrome浏览器会去掉前面的 `javascript` 字母，需要手动加上，部分浏览器的地址栏可能不支持输入脚本）。（如果你想快速体验，可选择这一种）

2. 复制脚本代码，然后新建浏览器书签，将代码粘贴到网址的输入框内，名字自己定义，然后保存。打开你需要操作的网页，然后再点击选择此书签。ok！（如果你以后也有使用的需求，可选择这一种）

3. 如果你会一点前端调试功能，可以直接打开网页的console，(chrome按F12键或右键页面任意处然后选择检查)。将复制的代码粘贴到console里，然后回车执行。（你想装逼？选择这种吧）

>刷新页面即失效
