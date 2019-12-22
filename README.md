# Hexo
Hexo blog

github.io访问地址：[Dawn](https://acdiost.github.io)
netlify.com访问地址：[Dawn](https://acdiost.netlify.com)
serverless访问地址：[Dawn](http://my-bucket-1300491156.cos-website.ap-guangzhou.myqcloud.com/)

## 常用操作
```
npm config set registry https://registry.npm.taobao.org
sudo npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo clean
hexo generate
hexo server

layout:[post, page, draft]
hexo new [layout] <title>
hexo publish [layout] <title>
hexo new photo "My Gallery"
```

## serverlsee部署
```
sudo npm install -g serverless
sudo chown -R $USER:$(id -gn $USER) /home/$USER/.config 
touch serverless.yml
## 添加本项目中serverless代码按自己的改
hexo g
serverless --debug
sls remove --debug
```

## github.io部署
```
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
4. 站点优化设置