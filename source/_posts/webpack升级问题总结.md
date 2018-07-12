---
title: VueJS升级问题总结
tags: [webpack升级, webpack]
---

## webpack1.x升级webpack2.x填坑总结

公司老旧项目改造，由原vue1.0升级为vue2.0，对应的webpack也由1.0升级为3.0。这篇文章包含了webpack升级的一些填坑之路，vue升级请参考vue官方给出的解决方案。

<!-- more -->

#### 0、 webpack.config.js升级
按照官方文档[迁移到新版本](https://doc.webpack-china.org/guides/migrating/)修改配置文件。如果是使用早期`vue-cli`生成的模板，还需要修改`build/utils.js`内的`cssLoader`，`generateLoader`方法。使最终生成的`module.rules`符合官方文档。


#### 1、 node.fs

`ERROR: Can't resolve 'fs' in doT`。` photo-sphere-viewer`模块内部调用了`doT`，而`doT`是个模板引擎，内部调用了`fs`模块，导致`webpack`打包时报错，需要在webpack.config.js加入:

```js
module.exports = {
	// ...
	node: {
		fs: 'empty'
	}
	// ...
}
```

#### 2、 `babel-eslint`升级
lint检查选择性升级至`7.2.3`

#### 3、`extract-text-webpack-plugin`升级
webpack1.x使用`1.0.1`，在webpack2.x里，该plugin会将其他loader名称转成一个绝对路径，比如
`css-loader`转为`/User/xxx/project/node_modules/_css-loader@0.23.1@css-loader/index.js`，导致loader失效。

#### 4、`stylus-loader`配置修改
webpack2.x官方不再支持为loader设置自定义属性了，`webpack1.x`内实现的导入公共`styl`文件的方法不再生效。如果不这么做，代码里若使用了variable、mixin之类的，会报`$variable(变量名) has no property $property(属性名)`，这其实就是公共variable没有导入的错误，`stylus-loader`无法找到对应variable。

```js
// 不再生效
module.exports = {
	// ...
	stylus: {
		 import: [
	      path.resolve(__dirname, '../styleguide/src/components/variable.styl'),
	      path.resolve(__dirname, '../styleguide/src/components/mixin.styl')
       ]
	}
	// ...
}
```

需要借助其他辅助插件实现。这里选择使用`stylus-resources-loader`。在`vue-loader`的配置里添加。

```js
module.exports = {
	// ...
	module: {
		rules: [
			{
				test: /\.vue$/,
				loader: 'vue-loader',
				options: {
					loaders: {
						// other loader ...
						stylus: [{
					      loader: 'stylus-resources-loader',
					      options: {
					        resources: [
					          path.resolve(__dirname, '../styleguide/src/components/variable.styl'),
					          path.resolve(__dirname, '../styleguide/src/components/mixin.styl')
					        ]
					      }
					    }]
					}
				}
			}
		]
	}
	
	// ...
}
```

如果使用vue-cli的webpack模板，只需在`build/utils.js`，修改`cssLoaders`。

```js
//...
return {
	// ...
	stylus: generateLoaders('stylus').concat({
	  loader: 'stylus-resources-loader',
	  options: {
	    resources: [
	      path.resolve(__dirname, '../styleguide/src/components/variable.styl'),
	      path.resolve(__dirname, '../styleguide/src/components/mixin.styl')
	    ]
	  }
	})
	//...
}	
```

#### 5、无法解析模块
如果在resolve.include指定了一个相对路径，如`path.join(__dirname, '../node_modules')`，那么业务代码里使用到的模块，会只在这个目录里查找依赖。

业务代码里使用了`d3`，这个模块被安装在`node_modules`里，`d3/index.js`里使用`import`导入了`d3/node_modules`内的一个模块`d3-selection`，现在的路径结构大概是这样的。
`project/node_modules/d3/node_modules/d3-selection/`。所以会报`can't resolve module in project/node_modules/`。

需要修改`resolve.include`为`resolve.include = [/node_modules/]`。

#### 6、`"path/to/bundle.[hashcode].js" uglifyjs invalid assignment`

经过webpack的处理，所有的文件都被打包成`bundle[.hashcode].js`和`vendor[.hascode].js`，`uglifyJs`会报告上述错误，原因是webpack2.x内置的`UglifyJsPlugin`不能正确处理es6语法，[issue](https://github.com/webpack-contrib/uglifyjs-webpack-plugin/issues/187)。需要使用`plugin处理`，另外也需要删减一些babel-preset。详情见上面issue。

