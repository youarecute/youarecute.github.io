

pwd 查看当前所在文件路径
ls  查看文件夹下文件
mkdir 创建文件夹
touch 创建文件
git status 查看文件夹状态
git add 添加文件

ls  查看文件夹下文件

mkdir 创建文件夹
touch 创建文件

git status 查看文件夹当前文件是否有新修改过的
git add 添加文件路径




（将github复制成文件）

第一步:

$ git clone

打开自己的github的源库,点击右边clone and download ,换ssh,复制路径ssh形式
在指令git clone 后面添加路径
确定后再输入yes确定

第二步:配置公钥

$ ssh-keygen -t rsa -C "邮箱地址"
三次回车

在我的电脑打开用户.....打开.ssh文件夹,以记事本形式打开id_rsa.pub

打开自己的github的用户设置,点击ssh and GPG keys ,添加公钥,随意命名,记事本id_rsa.pub
全复制到公钥中
再次执行git clone命令

git config --global user.email "邮箱地址"(第一次匹配新的电脑在需要秘钥的情况下使用)

git config --global user.name "用户名"(第一次匹配新的电脑在需要秘钥的情况下使用)

git commit

git push -u origin master(仓库是才创建,第一次上传文件时 要写全)

git push (第二次及以后 直接这样写)



git 命令
	git add 文件路径 让创建出来的文件告诉git仓库要添加

	git commit -m “要提交的内容是什么---提交日志” 将我们做的操作提交到本地git仓库

	git push -u origin master （第一遍这么写，目的是设置以后默认都是往master分支上提交代码）

	git pull    更新本地仓库，使本地和github统一

	echo "内容" >> 要添加到的文件名称

解决代码冲突

	git stash   入栈（撤回）

	git pull

	git stash pop   出栈


git add 

ls -all 展示隐藏文件

rm -rf .git/  强制删除 .git/ 隐藏文


git 多人合作

git init 初始化git仓库(.git文件夹生成)

git remote add origin git@github.com:kll082511/project.git     确认本地和远程仓库连接

刚开始初始化出来的git仓库是不知道推送的远端git仓库是谁,

所以需要该命令告诉他要跟哪儿个远程仓库连接

1.github 创建一个用于多人合作的git仓库

2.推送版本的代码,内容到我们的仓库中

3.默认情况下哪儿个账号创建的仓库,只有本人能提交东西上来,如果想要多人开发,需要邀请别人成为协作者

