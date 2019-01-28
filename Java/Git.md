Git
=====================================
     
branch
----------------
新建分支 git branch [branchName]         
新建并切换到该分支 git checkout -b [branchName]             
新建追踪远程分支master的分支并切换到该分支 git checkout -b [branchName] origin/master   
设置远程追踪分支 git branch -u origin/master foo, foo就会跟踪origin/master,如果当前救灾foo分支，可以直接执行git branch -u origin/master      

rebase
------------
取出一系列的提交记录，“复制”它们，然后在另一个地方逐个放下去，创建更线性的提交历史，代码库的提交历史更加清晰        
想要把master分支合并到bugFix上：定位到bugFix分支，执行git rebase master,bugFix分支的提交将会在master的最顶端，此时master还没有更新，如果要更新master，需要切换到master上，执行git rebase bugFix。      
 
HEAD
------------
指向当前分支上最近一次的提交记录，通常情况下指向分支名（如bugFix)            
git checkout [分支名|commitId]可以切换Head的指向      
移动HEAD: 在引用后面加操作符，^向上移动一个，～<num>向上移动多个，如～3       

如果是一个提交是合并而来的，有多个父提交，git checkout [分支名|commitId]~[数字]可以指定选择第几个父提交，默认是1

移动分支，让分支指向另一个提交:git branch -f master HEAD～3，会将master分支强制指向HEAD的第三级父提交      
  

撤销
---------------
git reset : 通过把分支记录回退几个提交记录来实现撤销改动。git reset HEAD~1,把本地指向的提交记录向上移动一个。
git revert : 撤销远程分支的改动，如果远程分支的提交是C2->C1，git revert HEAD，会在当前的提交记录C2后面多一个新提交C2‘，成为C2'->C2->C1，C2‘是用来撤销C2这个提交的，也就是说C2'的状态和C1相同，这时候把C2‘推到远程分支就可以撤销C2这个提交        


cherry-pick
---------------
把一些提交放到当前位置（HEAD）下面，git cherry-pick <commitId>...         
如果不知道commitId，使用git rebase -i，会打开一个UI界面，列出将要被复制到目标分支的备选提交记录，可以进行:1.调整提交记录顺序(鼠标拖放)2.删除不想要的提交3.合并提交


修改之前的提交
--------------------
git rebase -i 将提交重新排序，把要修改的commit放在最前面，改完后使用commit --amend，再用git rebase -i调回原来的顺序，最后把master移到修改的最前端       

上述方法可能会导致冲突，可以使用cherry-pick来把任意的提交跟在HEAD之后，然后使用commit --amend，然后继续使用cherry-pick把剩余的commit添加           

Tag
--------------------
git tag [标签名][commitId]，不指定commitId会用HEAD指向的位置        

describe
--------------------
git describe <ref> ref是任何能被Git识别成提交记录的引用，如果没有指定，默认是HEAD 
           
输出结果是 
<pre>\<tag\>\_\<numCommits\>\_g\<hash\>
tag表示离ref最近的标签，numCommit指的是这个ref和tag相差多少个提交记录，hash表示你给的ref表示的提交记录哈希值的前几位，当ref提交记录上有某个标签时，则只输出标签名称
</pre>           


fetch
--------------------
git fetch 拉取远程分支的改动到本地，下载到本地origin/master上，更新远程分支的指针，但并不会更新本地master分支，会下载所有的提交记录到各个远程分支        
git fetch <remote> <place> -> git fetch origin foo 到远程仓库的foo分支上，获取所有本地不存在的提交，放在本地的origin/foo上

git fetch <remote> <source>:<destination>,从远端的source分支拉取提交，更新本地分支destination，但destination不能是当前分支，如果destination不存在，则会创建分支     
git fetch origin :bar -> <source>为空，会在本地创建一个bar分支

pull
--------------------
git pull 相当于 git fetch; git merge origin/master 的缩写，会先把提交下载到本地，然后通过merge合并这次拉取的提交     
   

push
-------------------
push之前进行rebase:  git fetch; git rebase origin/master; git push。
简写：git pull --rebase; git push               
进行merge:  git fetch; git merge origin/master; git push。简写：git pull; git push              

git push <remote> <place> -> git push origin master 切换到本地仓库的master分支，获取所有提交，再到远程仓库origin中找到master分支，将远程仓库没有的提交记录添加上去,这种情况是来源和去向分支相同的情况       
如果不同的话git push origin <source>:<destination>，source是git可以识别的记录，可以是分支名，也可以是HEAD～1之类的 如果推送上去的destination不存在，则会在远程创建这个分支    
git push origin :side -> <source>为空，会删除远程的side分支

