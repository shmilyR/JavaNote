# 命令  
## Linux命令  
+ ls -l
+ ls -a 
查看文件的命令
+ cat id_rsa.pub
+ grep id_rsa.pub 根据索引信息查看
+ tail id_rsa.pub 从倒数第一行查看
+ vi 编辑文本
+ :q 退出
+ esc+:q+! ：强制退出  
+ vim 文件名 创建并打开文件
+ i 编辑文件
+ esc : wq enter 保存并退出
## git命令
+ git init 初始化仓库  
+ git add . 添加到仓库
+ git status 查看文件提交状态
+ git commit 提交到缓冲区
+ git push 提交到远程仓库
+ git remote add origin git@gitee.com:码云名/远程仓库.git
+ git branch 查看分支
+ git checkout -b feature 分支名 创建并进入分支
+ git checkout 切换分支
+ git merge 合并分支后面的分支到当前分支
+ git merge feature/分支名 
+ git reflog 查看版本号
+ git log 显示提交日志详细信息
+ git reset --hard 上一个版本号  回到之前的修改
+ pull和merge的区别：
    + pull是将远程端的文件或者代码拉取到本地，然后再本地进行修改
    + 合并分支。git是用C语言写成的，有指针的指向。各个分支就相当于指针。
+ git push origin feature/分支名 提交到分支上
+ git pull origin master 更新项目,自动合并
+ git fetch origin master 
+ pull和fetch的区别：
    + pull = merge + fetch
    + fetch : 不会进行自动合并。解决方案：  

数据库快照读  
λ表达式  
.war:  
数据库索引 提高效率 memory  
\G 格式化  
explain 分析一条SQL语句的性能有没有用到索引  
查询类型 select_type:SIMPLE  
唯一索引 组件索引 聚集索引 非聚集索引   
数据库的索引类型  
distinct用法
写项目时，最好不要使用*,目的是SQL优化  
数据库limit:  
SQL语句的运行顺序：from->where->select  
having用法  







