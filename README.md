1. git clone

git clone -b hexo git@github.com:FIRF27/FIRF27.github.io.git hexoBlog
 
2. npm install
 
3.. git 插件
npm install hexo-deployer-git --save

4. 主题插件
npm install hexo-theme-fluid --save

npm uninstall hexo-theme-landscape

5. 图片恢复
cp -r .backupImage/*  node_modules/hexo-theme-fluid/source/img
