[本项目demo] （https://diaotai.github.io/）

# 本项目是我根据hexo搭建的，具体流程上完搜索，网上有很多教程

## 说一下期间趟的几个坑：

1. 那些和我一样习惯于使用cnpm的小伙伴们，在下载hexo的时候不要使用cnpm，不要使用cnpm，不要使用cnpm，我刚开始因为这个炸的稀里哗啦，折腾了好久

2. 如果在执行 hexo deploy 后,出现 error deployer not found:github 的错误，那么使用这条命令  npm install hexo-deployer-git --save 然后再试试

3. 刚开始部署成功后，网页可能会延迟几十秒才能使用，在这段时间，你可能会看见一个404 Not Found，耐心等待就好（前提是你网址没输错）