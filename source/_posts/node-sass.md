---
title:  记一次node-sass安装不成功的解决
date: 2019-7-19 10:10:45
categories:
- 前端
tags:
- sass
---


### 结论：

> node-sass安装过程：在指定的npm源上下载node-sass包，该包src目录包含了node-sass的npm包的C实现，
下载完node-sass包后安装该包的dependencies 指定的依赖, 然后执行package.json里面指定的install脚本和postinstall脚本, 
install脚本检测是否存在编译好的二进制，没有的话到指定site下载的node-sass的二进制包，扩展名后缀为.node
postinstall脚本检测node-sass的二进制文件是否存在(带强制性参数--force时, 就是不管是否存在二进制都要编译C文件到二进制),不存在则使用node-gyp编译node-sass目录下的src目录，转到node-gyp脚本编译node-sass,
node-gyp编译node-sass前需要下载当前nodejs版本的头文件，用来编译node-sass的src目录下的C文件。编译成功则安装完成。

### install脚本获取下载二进制地址函数和获取本地二进制函数
```js
function getBinaryUrl() {
  var site = getArgument('--sass-binary-site') ||
             process.env.SASS_BINARY_SITE  ||
             process.env.npm_config_sass_binary_site ||
             (pkg.nodeSassConfig && pkg.nodeSassConfig.binarySite) ||  //可以配置在项目里面的package.json里面的nodeSassConfig字段
             'https://github.com/sass/node-sass/releases/download';

  return [site, 'v' + pkg.version, getBinaryName()].join('/');
}
```
```js
function getBinaryPath() {
  var binaryPath;

  if (getArgument('--sass-binary-path')) {
    binaryPath = getArgument('--sass-binary-path');
  } else if (process.env.SASS_BINARY_PATH) {
    binaryPath = process.env.SASS_BINARY_PATH;
  } else if (process.env.npm_config_sass_binary_path) {
    binaryPath = process.env.npm_config_sass_binary_path;
  } else if (pkg.nodeSassConfig && pkg.nodeSassConfig.binaryPath) { //可以配置在项目里面的package.json里面的nodeSassConfig字段
    binaryPath = pkg.nodeSassConfig.binaryPath;
  } else {
    binaryPath = path.join(getBinaryDir(), getBinaryName().replace(/_(?=binding\.node)/, '/'));
  }

  if (process.versions.modules < 46) {
    return binaryPath;
  }

  try {
    return trueCasePathSync(binaryPath) || binaryPath;
  } catch (e) {
    return binaryPath;
  }
}
```

使用远程服务器的二进制:
```sh
yarn config set sass-binary-site http://npm.taobao.org/mirrors/node-sass
# export SASS_BINARY_SITE=http://npm.taobao.org/mirrors/node-sass
yarn add node-sass

# 使用 --sass-binary-site
yarn add node-sass   --sass-binary-site=http://npm.taobao.org/mirrors/node-sass
```



使用本地的二进制: 
```sh
#export SASS_BINARY_PATH=/root/sass/linux-x64-67_binding.node
yarn config set sass_binary_path /root/sass/linux-x64-67_binding.node
yarn add node-sass

# 使用 --sass-binary-path
yarn add node-sass   --sass-binary-path=/root/sass/linux-x64-67_binding.node
```

node-sass二进制文件下载地址:https://github.com/sass/node-sass/releases

使用本地二进制可能会有版本不兼容的问题。使用npm install node-sass@4.9.0  --sass-binary-path=/root/sass/linux-x64-64_binding.node --verbose
```sh
> node-sass@4.9.0 postinstall /tmp/test/node_modules/node-sass
> node scripts/build.js

Binary found at /root/sass/linux-x64-64_binding.node
Testing binary
Binary has a problem: Error: The module '/root/sass/linux-x64-64_binding.node'
was compiled against a different Node.js version using
NODE_MODULE_VERSION 64. This version of Node.js requires
NODE_MODULE_VERSION 67. Please try re-compiling or re-installing
```

说明下载下载的二进制版本编译而来的nodejs的ABI版本跟当前系统nodejs的ABI版本不匹配。[ABI跟nodejs版本关系](https://nodejs.org/zh-cn/download/releases/)linux-x64-64_binding  64即是ABI版本（感觉跟chrome版本。。。。) ,需要下载的67的 , [下载地址](https://github.com/sass/node-sass/releases)

NODE_MODULE_VERSION 指的是 Node.js 的 ABI (application binary interface) 版本号，用来确定编译 Node.js 的 C++ 库版本，以确定是否可以直接加载而不需重新编译。在早期版本中其作为一位十六进制值来储存，而现在表示为一个整数。[来自](https://nodejs.org/zh-cn/download/releases/)


### 过程

错误信息
```
[4/4] Building fresh packages...
error /root/.jenkins/workspace/basp-web-client_master/node_modules/node-sass: Command failed.
Exit code: 1
Command: node scripts/build.js
Arguments: 
Directory: /root/.jenkins/workspace/basp-web-client_master/node_modules/node-sass
Output:
Building: /usr/local/bin/node /root/.jenkins/workspace/basp-web-client_master/node_modules/node-gyp/bin/node-gyp.js rebuild --verbose --libsass_ext= --libsass_cflags= --libsass_ldflags= --libsass_library=
gyp info it worked if it ends with ok
gyp verb cli [ '/usr/local/bin/node',
gyp verb cli   '/root/.jenkins/workspace/basp-web-client_master/node_modules/node-gyp/bin/node-gyp.js',
gyp verb cli   'rebuild',
gyp verb cli   '--verbose',
gyp verb cli   '--libsass_ext=',
gyp verb cli   '--libsass_cflags=',
gyp verb cli   '--libsass_ldflags=',
gyp verb cli   '--libsass_library=' ]
gyp info using node-gyp@3.8.0
gyp info using node@11.6.0 | linux | x64
gyp verb command rebuild []
gyp verb command clean []
gyp verb clean removing "build" directory
gyp verb command configure []
gyp verb check python checking for Python executable "python2" in the PATH
gyp verb `which` succeeded python2 /usr/bin/python2
gyp verb check python version `/usr/bin/python2 -c "import sys; print "2.7.5
gyp verb check python version .%s.%s" % sys.version_info[:3];"` returned: %j
gyp verb get node dir no --target version specified, falling back to host node version: 11.6.0
gyp verb command install [ '11.6.0' ]
gyp verb install input version string "11.6.0"
gyp verb install installing version: 11.6.0
gyp verb install --ensure was passed, so won't reinstall if already installed
gyp verb install version not already installed, continuing with install 11.6.0
gyp verb ensuring nodedir is created /root/.node-gyp/11.6.0
gyp verb created nodedir /root/.node-gyp
gyp http GET https://nodejs.org/download/release/v11.6.0/node-v11.6.0-headers.tar.gz
gyp WARN install got an error, rolling back install
gyp verb command remove [ '11.6.0' ]
gyp verb remove using node-gyp dir: /root/.node-gyp
gyp verb remove removing target version: 11.6.0
gyp verb remove removing development files for version: 11.6.0
gyp ERR! configure error 
gyp ERR! stack Error: connect ETIMEDOUT 104.20.22.46:443
gyp ERR! stack     at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1082:14)
gyp ERR! System Linux 3.10.0-693.el7.x86_64
gyp ERR! command "/usr/local/bin/node" "/root/.jenkins/workspace/basp-web-client_master/node_modules/node-gyp/bin/node-gyp.js" "rebuild" "--verbose" "--libsass_ext=" "--libsass_cflags=" "--libsass_ldflags=" "--libsass_library="
gyp ERR! cwd /root/.jenkins/workspace/basp-web-client_master/node_modules/node-sass
gyp ERR! node -v v11.6.0
gyp ERR! node-gyp -v v3.8.0
gyp ERR! not ok 
Build failed with error code: 1
info Visit https://yarnpkg.com/en/docs/cli/install for documentation about this command.
Build step 'Execute shell' marked build as failure
Finished: FAILURE
```

1. 尝试104.20.22.46  在线域名反向找到域名地址. 失败...
2. ping nodejs.org  。 IP地址是 104.20.22.46  真巧... , node_modules搜索nodejs.org....
3. 检查服务器联网情况，pign 104.20.22.46   / ping npm.taobao.org   超时... 
3. google "stack Error: connect ETIMEDOUT 104.20.22.46:443" 是某一C模块问题.  一脸懵逼中。。。
4.  "Exit code: 1
Command: node scripts/build.js"  ????  查看node-sass package.json 查看scripts字段 包含install和postinstall

```json
{
   "scripts": {
    "coverage": "node scripts/coverage.js",
    "install": "node scripts/install.js",
    "postinstall": "node scripts/build.js",
    "lint": "node_modules/.bin/eslint bin/node-sass lib scripts test",
    "test": "node_modules/.bin/mocha test/{*,**/**}.js",
    "build": "node scripts/build.js --force",
    "prepublish": "not-in-install && node scripts/prepublish.js || in-install"
  }
}
```

查看scripts/build.js 

```js
function testBinary(options) {
  if (options.force || process.env.SASS_FORCE_BUILD) {
    return build(options);
  }
// 存在二进制？？？
  if (!sass.hasBinary(sass.getBinaryPath())) {
    return build(options);
  }

  console.log('Binary found at', sass.getBinaryPath());
  console.log('Testing binary');

  try {
      //测试二进制有效???
    require('../').renderSync({
      data: 's { a: ss }'
    });

    console.log('Binary is fine');
  } catch (e) { 
    console.log('Binary has a problem:', e);
    console.log('Building the binary locally');
     // 糟糕的二进制，还是要编译....
    return build(options);
  }
}

testBinary(parseArgs(process.argv.slice(2)));
``` 

查看scripts/install.js
```js
function checkAndDownloadBinary() {
  if (process.env.SKIP_SASS_BINARY_DOWNLOAD_FOR_CI) {
    console.log('Skipping downloading binaries on CI builds');
    return;
  }

  var cachedBinary = sass.getCachedBinary(),
    cachePath = sass.getBinaryCachePath(),
    binaryPath = sass.getBinaryPath();  // 获取配置的二进制文件路径

  if (sass.hasBinary(binaryPath)) {  // 配置了二进制文件路径咱们就不下载了。。
    console.log('node-sass build', 'Binary found at', binaryPath);
    return;
  }

  try {
    mkdir.sync(path.dirname(binaryPath));
  } catch (err) {
    console.error('Unable to save binary', path.dirname(binaryPath), ':', err);
    return;
  }

  if (cachedBinary) { // 这是安装后执行第二次同样的安装...
    console.log('Cached binary found at', cachedBinary);
    fs.createReadStream(cachedBinary).pipe(fs.createWriteStream(binaryPath));
    return;
  }

  // 没有二进制乖乖下载吧。。。。。 getBinaryUrl解析了怎么拿到下载地址。。。
  download(sass.getBinaryUrl(), binaryPath, function(err) {
    if (err) {
      console.error(err);
      return;
    }

    console.log('Binary saved to', binaryPath);

    cachedBinary = path.join(cachePath, sass.getBinaryName());

    if (cachePath) {
      console.log('Caching binary to', cachedBinary);

      try {
        mkdir.sync(path.dirname(cachedBinary));
        fs.createReadStream(binaryPath)
          .pipe(fs.createWriteStream(cachedBinary))
          .on('error', function (err) {
            console.log('Failed to cache binary:', err);
          });
      } catch (err) {
        console.log('Failed to cache binary:', err);
      }
    }
  });
}
 // If binary does not exist, download it
checkAndDownloadBinary();
```

最后回到上面的getBinaryPath和getBinaryUrl函数....


> tip 最后： sass已经出了dart写的版本dart-sass,新项目建议安装使用npm install sass --save-dev ,避免不必要的麻烦。。。
