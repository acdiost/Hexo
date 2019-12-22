# Hexo
Hexo blog

访问地址：[Dawn](https://acdiost.netlify.com)

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
sudo chown -R $USER:$(id -gn $USER) /home/dawn/.config 
touch serverless.yml
hexo g
serverless --debug
sls remove --debug
```
