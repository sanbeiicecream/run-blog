---
title: Firefox自定义样式
date: 2024-01-21
last: 
draft: 
category: []
tags: [浏览器]
series: []
cover:
  image: "http://www.firefox.com.cn/media/img/favicons/firefox/browser/favicon-196x196.59e3822720be.png"
  alt: ""
  caption: ""
---

## 实时调试
火狐浏览器给用户开放了浏览器窗口样式调试的方法，可以像调试网页那样调试窗口样式  

在 Win/Linux 上按 `Ctrl + Shift + I` 或在 Mac 上按 `Cmd + Opt + I`
![image.png](https://image.jysgdyc.top:443/blog/20240121222248.png)

![image.png](https://image.jysgdyc.top:443/blog/20240121222258.png)





## 火狐隐藏工具栏
导航栏输入`about:config`  
将toolkit.legacyUserProfileCustomizations.stylesheets更改为 true  

导航栏输入`about:profiles`  

![image.png](https://image.jysgdyc.top:443/blog/20240121220806.png)

打开文件夹
1. 创建 chrome 文件夹
2. 新建 userChrome.css 文件，（用记事本改成css文件即可）

```css
/* Source file https://github.com/MrOtherGuy/firefox-csshacks/tree/master/chrome/hide_tabs_toolbar.css made available under Mozilla Public License v. 2.0
See the above repository for updates as well as full license text. */

/* Hides tabs toolbar */

/* For OSX use hide_tabs_toolbar_osx.css instead */

/* Note, if you have either native titlebar or menubar enabled, then you don't really need this style.
 * In those cases you can just use: #TabsToolbar{ visibility: collapse !important }
 */
 
/* IMPORTANT */
/*
Get window_control_placeholder_support.css
Window controls will be all wrong without it
*/


:root[tabsintitlebar] {
  --uc-toolbar-height: 40px;
}

:root[tabsintitlebar][uidensity="compact"] {
  --uc-toolbar-height: 32px
}

#TabsToolbar {
  visibility: collapse !important
}

:root[sizemode="fullscreen"] #TabsToolbar> :is(#window-controls, .titlebar-buttonbox-container) {
  visibility: visible !important;
  z-index: 2;
 }

:root:not([inFullscreen]) #nav-bar {
  margin-top: calc(0px - var(--uc-toolbar-height, 0px));
}

:root[tabsintitlebar] #toolbar-menubar[autohide="true"] {
  min-height: unset !important;
  height: var(--uc-toolbar-height, 0px) !important;
  position: relative;
}

#toolbar-menubar[autohide="false"] {
  margin-bottom: var(--uc-toolbar-height, 0px)
}

:root[tabsintitlebar] #toolbar-menubar[autohide="true"] #main-menubar {
  flex-grow: 0;
  align-items: stretch;
  background-color: var(--toolbar-bgcolor, --toolbar-non-lwt-bgcolor);
  background-clip: padding-box;
  border-right: 30px solid transparent;
  border-image: linear-gradient(to left, transparent, var(--toolbar-bgcolor, --toolbar-non-lwt-bgcolor) 30px) 20 / 30px

}

#toolbar-menubar:not([inactive]) {
  z-index: 2
}

#toolbar-menubar[autohide="true"][inactive]>#menubar-items {
  opacity: 0;
  pointer-events: none;
  margin-left: var(--uc-window-drag-space-pre, 0px)
}

#nav-bar {
  margin-right: 138px;
}

.titlebar-buttonbox-container {
  background: #fff;
}

/* 增加 窗口缩小时，需要空白进行拖动 */
#nav-bar-customization-target {
  margin-left: 1.2rem;
}

/* 增加 窗口缩小时，需要空白进行拖动 */
.titlebar-buttonbox-container {
  margin-right: 1.2rem;
}
```

