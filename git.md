### 本地向github推送代码
#### 1.1创建本地仓库，并与远程仓库建立连接
1. 项目文件名右键选择Git Bash Here打开Git客户端
2. 输入命令：git init 
    生成.git文件
3. git add .
    将项目中的文件添加到本地仓库
4. git commit -m "提交备注"
    将项目中的文件提交到本地仓库
5. git remote add origin https://github.com/Marin-git/...
    将本地仓库与远程仓库建立连接
6. git push -u origin master
    将本地文件推送到GitHub上

#### 1.2更新文件
1. git add .
2. git commit -m "..."
3. git pull origin master
4. git push -u origin master