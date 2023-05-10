# git上传代码

`

### 1. 创建远程仓库
1.  打开你的git，选择新建仓库
	![img](https://github.com/LingYe-WEB3/secure/blob/f456fa010c928145a516ed01b9bcf46e169285a7/git/image/Pasted%20image%2020230510111306.png)
2. 
	填写仓库名称
	仓库的可见性
	是否创建readme.md文件
	点击创建
	![img](https://github.com/LingYe-WEB3/secure/blob/eed589c47c8f8ad4978091db70824a1121a4a3e1/git/image/Pasted%20image%2020230510111548.png)

3. 远程仓库创建成功，这里显示的是你的仓库地址。
	![img](https://github.com/LingYe-WEB3/secure/blob/eed589c47c8f8ad4978091db70824a1121a4a3e1/git/image/Pasted%20image%2020230510111902.png)



### 2. 创建远程仓库后第一次上传代码

1.  创建一个文件夹为本地仓库，用git bash here打开本地文件夹
    ![[Pasted image 20230510112212.png]]

2.  添加你要上传的内容，比如下面这句话就是在本地仓库中创建了一个markdown格式的文件，命名为readme。内容为: # test
	`echo "# test" >> readme.md`
3.  初始化本地仓库
	`git init`
4. 链接到远程仓库
	`git remote add origin https://github.com/LingYe-WEB3/test.git`
5. 创建分支
	`git branch -M main`
1. 添加要上传的内容
	`git add *`
6. 添加提交的注释
	`git commit -m "first commit"`
7. 从本地推送到远程仓库
	`git push -u origin main`


![[Pasted image 20230510135416.png]]


刷新仓库，发现文件已经成功上传
![[Pasted image 20230510135526.png]]


### 3. 后续上传代码：

首先要把远程仓库的代码同步到本地
![[Pasted image 20230510142610.png]]


然后做以下操作新增代码到远程：
1. 在文件中放入你新增的代码
2. 连接到远程仓库
3. 添加要上传的内容
4. 添加提交的注释
5. 提交到远程仓库
```
mkdir test1
echo "test1234" >> test1/test.md

git remote add origin https://github.com/LingYe-WEB3/test.git

git add *

git commit -m "second commit"

git push origin main
```

![[Pasted image 20230510140804.png]]
刷新仓库，查看新增的文件
![[Pasted image 20230510141028.png]]


** 如果已经链接到了远程仓库，可以通过`git remote rm origin`重新进行连接。

跟新本地仓库文件（覆盖写入）
```
git fetch --all
git pull 
git reset --hard origin main
```

跟新本地仓库，合并写入
```
git stach
git pull
git stach pop
```