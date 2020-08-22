# Hexo

acdiost`s hexo blog

github.io访问地址：[Dawn](https://acdiost.github.io)

netlify.com访问地址：[Dawn](https://acdiost.netlify.com)

serverless访问地址：[Dawn](http://my-bucket-1300491156.cos-website.ap-guangzhou.myqcloud.com/)

midway-fass访问地址: [Dawn](http://test1yi7vczwwpgo1vdijvn.workbenchapi.com/)

## 常用操作

```bash
curl -sL https://rpm.nodesource.com/setup_lts.x | sudo bash -
yum install nodejs -y
sudo npm config set registry https://registry.npm.taobao.org
sudo npm config set registry https://registry.npmjs.org
npm get registry
sudo npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo clean
hexo generate
hexo server
hexo deploy

layout:[post, page, draft]
hexo new [layout] <title>
hexo publish [layout] <title>
hexo new photo "My Gallery"
```

## serverless部署

```bash
sudo npm install -g serverless
sudo chown -R $USER:$(id -gn $USER) /home/$USER/.config
touch serverless.yml
## 添加本项目中serverless代码按自己的改
hexo g
# 开始部署
serverless --debug
# 移除部署
sls remove --debug

# 计费规则
https://cloud.tencent.com/document/product/436/16871
# 控制台管理
https://console.cloud.tencent.com/cos5
# 配置域名
https://cloud.tencent.com/document/product/436/36638
```

## github.io部署

```bash
# 项目名称是github.io结尾
sudo npm install --save hexo-deployer-git
修改_config.yml文件
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:acdiost/acdiost.github.io.git
  branch: master
hexo d
```

## netlify.com部署

1. 注册用户
2. new site from git 选择当前项目
3. 点击部署

* 非静态页面需添加部署命令:`hexo generate`
* 指定静态文件目录为:`./public`

4. 站点优化设置

## Midway Serverless部署

> 参考文档：<https://help.aliyun.com/document_detail/172145.html>

1. （建议直接用它默认的可忽略下面操作，直接部署即可）上传代码至codeup <https://codeup.aliyun.com/>
2. 进入云开发平台 <https://workbench.aliyun.com/application> 创建hexo应用  
    新建应用--> 实验室--> 解决方案(serverless hexo)-->填写信息
3. 选择刚刚新建应用里的应用配置,在全部环境中添加一个域名(需备案)
4. 编辑 `package.json` 新增依赖

  ```json
    "dependencies": {
      "@midwayjs/decorator": "^2.0.16",
      "@midwayjs/faas": "^0.3.0",
      "@midwayjs/faas-middleware-static-file": "^0.0.3",
      hexo依赖...
    }
  ```

5. 新增配置文件 `f.yml`

  ```yml
service
  name: serverless-hexo
provider:
  name: aliyun
  runtime: nodejs10
functions:
  render:
    handler: render.handler
    events:
      - apigw:
          path: /*
package:
  include:
    - public
  artifact: code.zip
  ```

6. 点击开发部署进入,等待页面加载完成,在终端中执行`npm install`
7. 点击左上角第一个选项,选择相应的环境，新增一个路由  
   文件同步填写`/*` 路由填写:`render.handler` 方法选择`get`点击部署.  
   线上环境需先在预发环境部署
