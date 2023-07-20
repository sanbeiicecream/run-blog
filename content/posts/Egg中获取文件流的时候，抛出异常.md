---
title: "Egg中获取文件流的时候，抛出异常"
date: "2023-07-20T17:52:23.717Z"
draft: 
category: [] 
tags: []
series: []
cover: 
    image: 'https://image.jysgdyc.top:443/blog/20230721015409.png'
    alt: ''
    caption: ''
---
在使用Egg作为服务端，上传**jfif文件**时报了个错  
```js
ctx.getFileStream()
```

![image.png](https://image.jysgdyc.top:443/blog/20230721015409.png)  

这次抱着深究源码的态度进行解决问题，进行查看相关关键代码  
## 源码
```js
// node_modules\.pnpm\egg-multipart@3.3.0\node_modules\egg-multipart\app.js
// 插件入口
'use strict';

const { normalizeOptions } = require('./lib/utils');

module.exports = class AppBootHook {
  constructor(app) {
    this.app = app;
  }

  configWillLoad() {
  // 配置初始化
    this.app.config.multipart = normalizeOptions(this.app.config.multipart);
    const options = this.app.config.multipart;

    this.app.coreLogger.info('[egg-multipart] %s mode enable', options.mode);
    if (options.mode === 'file' || options.fileModeMatch) {
      this.app.coreLogger.info('[egg-multipart] will save temporary files to %j, cleanup job cron: %j', options.tmpdir, options.cleanSchedule.cron);
      // enable multipart middleware
      this.app.config.coreMiddleware.push('multipart');
    }
  }
};
```

```js
//node_modules\.pnpm\egg-multipart@3.3.0\node_modules\egg-multipart\lib\utils.js
// 内置文件名白名单
exports.whitelist = [
  // images
  '.jpg', '.jpeg', // image/jpeg
  '.png', // image/png, image/x-png
  '.gif', // image/gif
  '.bmp', // image/bmp
  '.wbmp', // image/vnd.wap.wbmp
  '.webp',
  '.tif',
  '.psd',
  // text
  '.svg',
  '.js', '.jsx',
  '.json',
  '.css', '.less',
  '.html', '.htm',
  '.xml',
  // tar
  '.zip',
  '.gz', '.tgz', '.gzip',
  // video
  '.mp3',
  '.mp4',
  '.avi',
];

exports.normalizeOptions = options => {
  //...
    // normalize whitelist
  if (Array.isArray(options.whitelist)) options.whitelist = options.whitelist.map(extname => extname.toLowerCase());

  // normalize fileExtensions
  if (Array.isArray(options.fileExtensions)) {
    options.fileExtensions = options.fileExtensions.map(extname => {
      return (extname.startsWith('.') || extname === '') ? extname.toLowerCase() : `.${extname.toLowerCase()}`;
    });
  }
  //...
  function checkExt(fileName) {
    if (typeof options.whitelist === 'function') return options.whitelist(fileName);
    const extname = path.extname(fileName).toLowerCase();
    if (Array.isArray(options.whitelist)) return options.whitelist.includes(extname);
    // 或条件，存在白名单中或者在extname配置中
    return exports.whitelist.includes(extname) || options.fileExtensions.includes(extname);
  }

  options.checkFile = (fieldName, fileStream, fileName) => {
    // just ignore, if no file
    if (!fileStream || !fileName) return null;
    try {
	  // 文件名是否允许上传的判断，不可以就抛出异常
      if (!checkExt(fileName)) {
        const err = new Error('Invalid filename: ' + fileName);
        err.status = 400;
        return err;
      }
    } catch (err) {
      err.status = 400;
      return err;
    }
  };
}
```

```js
// node_modules\.pnpm\egg-multipart@3.3.0\node_modules\egg-multipart\app\extend\context.js
const parse = require('co-busboy');
//...
module.exports = {
  // 创建multipart.parts实例
  multipart(options) {
    //...
    const { autoFields, defaultCharset, defaultParamCharset, checkFile } = ctx.app.config.multipart;
    const { fieldNameSize, fieldSize, fields, fileSize, files } = ctx.app.config.multipart;
    options = extractOptions(options);
    const parseOptions = Object.assign({
      autoFields,
      defCharset: defaultCharset,
      defParamCharset: defaultParamCharset,
      checkFile,
    }, options);

    // https://github.com/mscdex/busboy#busboy-methods
    // merge limits
    parseOptions.limits = Object.assign({
      fieldNameSize,
      fieldSize,
      fields,
      fileSize,
      files,
    }, options.limits);
    const parts = parse(this, parseOptions);
  },
  // egg中获取上传文件流的方法
  async getFileStream(options = {}) {
    options.autoFields = true;
    // 调用上面函数
    const parts = this.multipart(options);
    let stream = await parts();
    //...
  }
}

```

```js
// node_modules\.pnpm\co-busboy@2.0.0\node_modules\co-busboy\index.js
// 用于解析传入的 HTML 表单数据
var Busboy = require('busboy')
...
var busboy = Busboy(options)
var checkFile = options.checkFile
...
request = inflate(request)
request.on('close', cleanup)

busboy
  .on('field', onField)
  .on('file', onFile)
  .on('close', cleanup)
  .on('error', onEnd)
  .on('finish', onEnd)

// 解析文件流类型
function onFile(fieldname, file, info) {
  var filename = info.filename
  var encoding = info.encoding
  var mimetype = info.mimeType
  if (checkFile) {
    // 关键，查看文件流是否符合条件
    var err = checkFile(fieldname, file, filename, encoding, mimetype)
    if (err) {
      // make sure request stream's data has been read
      var blackHoleStream = new BlackHoleStream()
      file.pipe(blackHoleStream)
      return onError(err)
    }
  }

  // opinionated, but 5 arguments is ridiculous
  file.fieldname = fieldname
  file.filename = filename
  file.transferEncoding = file.encoding = encoding
  file.mimeType = file.mime = mimetype
  ch(file)
}

```

[busboy](https://github.com/mscdex/busboy)  
根据上面代码可以看出ctx.getFileStream()使用了内置的egg-multipart插件  
执行流程大概是先使用normalizeOptions进行egg的multipart配置初始化，后续在创建的multipart.parts实例中对流进行判断处理。其中使用checkFile对文件流判断，**报错原因就是因为文件名称没有存在whitelist或extname配置中**  
## 解决办法  
```js
// app\config\config.default.js

module.exports = appInfo => {
    config.multipart = {
    fileExtensions: ['jfif'], // 需要上传文件的后缀，如果存在whitelist中可以不添加
  };
}
```

## 总结
观看源码可以直奔主题，看的是实现思路，而不需要把每行代码都看懂，这样可以加快源码阅读的速度。