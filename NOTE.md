# 本地搭建react源码调试环境

- 使用版本: 17.0.2

1. 从git上clone源码

`git clone https://github.com/facebook/react.git`

2. 在本地创建项目

    ```
    yarn create react-app learn-react-sourcecode
    或
    npm create-react-app learn-react-sourcecode
    ```

3. 创建项目后运行`yarn eject或npm eject`暴露eject配置

4. 引入react/packages

   在learn-react-sourcecode/src新建react目录，把第一步下载的react源码中的packages复制到src/react目录下。

5. 修改learn-react-sourcecode/config/webpack.config.js配置

    ```
    // ...
    resolve: {
      alias: {
        'react-native': 'react-native-web',
        'react': path.resolve(__dirname, '../src/react/packages/react'),
        'react-dom': path.resolve(__dirname, '../src/react/packages/react-dom'),
        'shared': path.resolve(__dirname, '../src/react/packages/shared'),
        'react-reconciler': path.resolve(__dirname, '../src/react/packages/react-reconciler'),
        //'react-events': path.resolve(__dirname, '../src/react/packages/events')
      }
    }
    ```

    如果版本是react16.8.6之前的，需要添加react-events配置，同时还需要将src/react/packages中所有的import xxx from 'events'改成from 'react-events'。我这里使用的是17.0.2版本，所以不需要。

6. 修改环境变量 config/env.js。设置全局变量和对flow类型判断进行处理，减少文件的报错提示

    ```
    const stringified = {
      __DEV__: true,
      __PROFILE__: true,
      __UMD__: true,
      __EXPERIMENTAL__: true,
      'process.env': Object.keys(raw).reduce((env, key) => {
        env[key] = JSON.stringify(raw[key])
        return env
      }, {})
    }
    ```
   
   根目录下创建`.eslintrc.json`文件
   ```
   {
      "extends": "react-app",
      "globals": {
        "__DEV__": true,
        "__PROFILE__": true,
        "__UMD__": true,
        "__EXPERIMENTAL__": true   
      }
    }
   ```

   添加flow类型判断
   `yarn add @babel/plugin-transform-flow-strip-types -D`

   在config/webpack.config.js中babel-loader的plugin添加配置
   ```
   plugins: [
      require.resolve('@babel/plugin-transform-flow-strip-types'),
      // ...
    ],
   ```

7. 导出HostConfig
   修改src/packages/react-reconciler/src/ReactFiberHostConfig.js导出HostConfig

   ```
   // 注释掉原来的
   // import invariant from 'shared/invariant';
   // invariant(false, 'This module must be shimmed by a specific renderer.');

   // 添加
   export * from './forks/ReactFiberHostConfig.dom';
   ```

   修改 `src/react/packages/shared/ReactSharedInternals.js`
   ```
   // 注释掉
   // import React from 'react';
   // 注释掉
   // const ReactSharedInternals =
   // React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED;
   // 新增
   import ReactSharedInternals from '../react/src/ReactSharedInternals';
   ```

8. 添加项目vscode配置
   vscode里搜索 Flow Language Support 并且安装。
   根目录下新建 .vscode/settings.json
   ```
   "javascript.validate.enable": false,
   "typescript.validate.enable": false,
   ```

9. 测试

src/index.js

```
import * as React from 'react';
import * as ReactDOM from 'react-dom';
```

运行 `yarn start`。会报错，缺少 `@babel/plugin-syntax-jsx`模块。

安装: `yarn add @babel/plugin-syntax-jsx -D`

再次运行，会报eslint相关的错误。很多很多。如果不想改的话，直接在config/webpack.config.js中将eslint-plugin插件的配置注释掉。

[参考地址](https://zhuanlan.zhihu.com/p/336933386)
