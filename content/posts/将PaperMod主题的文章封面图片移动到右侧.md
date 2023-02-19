---
title: "将PaperMod主题图片移动到右侧"
date: "2023-02-19T09:51:19.864Z"
draft: 
category: [] 
tags: [建站]
series: []
cover: 
    image: 'https://image.jysgdyc.top:443/blog-images/wallhaven-qz2d65.png'
    alt: '替换文本'
    caption: '封面标题'
---
![image.png](https://image.jysgdyc.top:443/blog-images/20230219175857.png)

上面图片占用的面积太大了，需要调整一下放在右侧边
修改文件layouts => _default => list.html
定位到关键代码
``` html
  <article class="{{ $class }}">
    {{- $isHidden := (site.Params.cover.hidden | default site.Params.cover.hiddenInList) }}
    {{- partial "cover.html" (dict "cxt" . "IsHome" true "isHidden" $isHidden) }}
    <header class="entry-header">
      <h2>
        {{- .Title }}
        {{- if .Draft }}<sup><span class="entry-isdraft">&nbsp;&nbsp;[draft]</span></sup>{{- end }}
      </h2>
    </header>
    {{- if (ne (.Param "hideSummary") true) }}
    <div class="entry-content">
      <p>{{ .Summary | plainify | htmlUnescape }}{{ if .Truncated }}...{{ end }}</p>
    </div>
    {{- end }}
    {{- if not (.Param "hideMeta") }}
    <footer class="entry-footer">
      {{- partial "post_meta.html" . -}}
    </footer>
    {{- end }}
    <a class="entry-link" aria-label="post link to {{ .Title | plainify }}" href="{{ .Permalink }}"></a>
  </article>
```
修改页面dom结构
``` html
  <article class="{{ $class }} list-container">
    <div>
      <header class="entry-header">
        <h2>
          {{- .Title }}
          {{- if .Draft }}<sup><span class="entry-isdraft">&nbsp;&nbsp;[draft]</span></sup>{{- end }}
        </h2>
      </header>
      {{- if (ne (.Param "hideSummary") true) }}
      <div class="entry-content">
        <p>{{ .Summary | plainify | htmlUnescape }}{{ if .Truncated }}...{{ end }}</p>
      </div>
      {{- end }}
      {{- if not (.Param "hideMeta") }}
      <footer class="entry-footer">
        {{- partial "post_meta.html" . -}}
      </footer>
      {{- end }}
      <a class="entry-link" aria-label="post link to {{ .Title | plainify }}" href="{{ .Permalink }}"></a>
    </div>
    <div class="list-pic">
      {{- $isHidden := (site.Params.cover.hidden | default site.Params.cover.hiddenInList) }}
      {{- partial "cover.html" (dict "cxt" . "IsHome" true "isHidden" $isHidden) }}
    </div>
  </article>
```
在blank.css中添加
``` css
  .list-container{
    display: flex;
  }
  .list-pic{
    width: 38%;
  }
```
完成！
![image.png](https://image.jysgdyc.top:443/blog-images/20230219175939.png)
