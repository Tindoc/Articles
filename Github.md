# Git 相关
> 免责声明：以下内容未经严格测试，请勿在任何真实环境中使用，出现问题概不负责

## 参考资料
- [Learn git branching](https://oschina.gitee.io/learn-git-branching/)

    闯关形式学习 git。不要单纯为了实现右侧图形，假想场景练习效果更好
- [Pro Git](https://gitee.com/progit/)
    
    非官方链接，但文档排版合适，适合用来详细学习（其中，5.2节提供了详细的协作案例）
    > 小缺陷：代码部分的第二行开始增加了不必要的缩进
- [Git 官网](https://git-scm.com/)

## Git
### 1. 基本概念
- HEAD

    Git 保存着一个名为 HEAD 的 **特别指针** 用来识别当前在哪个分支上工作。在 Git 中，它是一个指向你正在工作中的本地分支的指针（可以将 HEAD 想象为当前分支的别名）
- 暂存区 stage

- 工作目录 working tree

- 分支 branch

    Git 中的分支，本质上仅仅是个指向 commit 对象的 **可变指针**（分支还可以理解为从某个提交对象往回看的历史）
    > 在 Git 中分支实际上仅是一个包含所指对象校验和（40 个字符长度 SHA-1 字串）的文件
- 分离 HEAD 状态

    HEAD 不指向当前分支的状态（即 HEAD 指针和分支指针指向不同）
- 远程仓库追踪 remote tracking

    master 和 origin/master 的关联关系就是由分支的 **remote tracking** 属性决定的。master 被设定为跟踪 origin/master（这意味着为 master 分支指定了推送的目的地以及拉取后合并的目标）

### 2. Branch 分支
#### 2.1 创建分支
- ` git branch <branchName> ` 
    创建一个名为 branchName 的分支
- ` git checkout -b <branchName> ` 
    创建完分支之后马上切换到该分支
- ` git branch <branchName> [ref] `
    在指定的 ref 上创建分支（ref 不是必须，如果没有则在当前 HEAD 创建）
- 较少使用
    - ` git branch -f <branchName> <ref> `
        将 **非当前分支** branchName 分支强制（-f）移动到 ref 的位置，相当于在 branchName 分支上执行` git reset <ref> `，优势：可以不用先切换到 branchName 分支
    - ` git branch nameSpace/branchName `
        创建一个名为 nameSpace/branchName 的分支
        其中，Git 支持把分支名称分置于不同命名空间下，命名空间可以用来指明提交者，例如，zhangsan/featureA

> 如果创建分支前暂存区或工作目录中包含内容，创建的分支上也会保留这些内容，且和原分支上的相互独立，进行后续操作不会影响到原分支（因为刚刚创建，这两个分支的暂存区或工作目录中包含的内容相同，所以这个时候相互切换不会导致冲突而阻止切换）
@Q,真的相互独立？貌似另一分支提交之后原分支的就不在了
#### 2.2 切换分支
` git checkout xxxBranch `

注意：切换分支前如果暂存区或工作目录内包含尚未提交的修改，如果这些修改与即将检出的分支不会产生冲突则会直接将这些修改带到切换的分支上，如果会与即将检出的分支产生冲突则会中止切换。**建议：切换分支前最好保持一个清洁的工作区域**。

> Git 提示如下：
> 
> ``` shell
> error: Your local changes to the following files would be overwritten by checkout:
>     xxxxx.xxx
>     xxxxx.xxx
> Please commit your changes or stash them before you switch branches.
> Aborting
> ```
> 
> 解决方法：使用 stashing 或 commit amending
#### 2.3 整合分支
两种方式：
1. 合并（merge）
2. 衍合（rabase）
##### 2.3.1 分支合并（merge）
> [git merge 文档翻译](https://www.jianshu.com/p/58a166f24c81)
1. 不发生冲突时的 **快进合并**
    前提：当前分支为 master，执行` git merge aBranch `
    如果顺着一个分支（master）走下去可以到达另一个分支（aBranch）的话，那么 Git 在合并两者时，只会简单地把指针右移，因为 **这种单线的历史分支不存在任何需要解决的分歧**，这种合并过程称为快进（Fast forward）
    > “顺着一个分支（master）走下去可以到达另一个分支（aBranch）” 这句话怎么理解？新建一个分支简单来说就是在当前 HEAD 上新建一个分支指针（前提条件：没有显式指定在哪里创建分支，可以使用` git branch branchName [ref] `指定非当前 HEAD），新建分支后只在新建的分支上做提交，没有再在 master 分支上做任何提交，这种情况下，新建分支上的提交记录是直接连在 master 分支的提交记录链上的，即所说的可以从 master 分支走到新建分支。这个时候合并新建分支到 master 分支，只需要简单移动 master 分支的指针即可。
    >
    > 这种 “分支” 操作是 Git 的一大特色。

    > 建议：尽量使用` --no-ff `来避免快进合并模式，每次合并都创建一个提交记录
2. 遇到冲突时的分支合并
    **分支合并遇到冲突时，Git 先做合并，但不提交，它会停下来等你解决冲突**。

    解决冲突的步骤：
    1. 使用` git status `查看哪些文件在合并时发生了冲突，这些文件会在 **Unmerged paths:** 下列出
    2. 在解决了所有文件的冲突后，运行` git add `将把它们标记为已解决状态（实际上就是来一次快照保存到暂存区域）。**因为一旦暂存，就表示冲突已经解决**（第1步执行后的提示中有说明使用` git add `命令）
    3. 确认所有冲突都已解决，也就是进入了暂存区，就可以用` git commit `来完成这次合并提交（注意，这会产生一个提交记录）
    
    > 可以使用` git merge --abort `中止此次合并，第1步执行后的提示中有该命令的提示）
3. 其他
    ` git merge --no-commit --squash aBranch `
    - ` --squash `选项将目标分支（即 aBranch）上的所有更改全拿来应用到当前分支上（类似于使用` git cherry-pick `将 aBranch 分支有而当前分支没有的提交记录全部添加到当前分支）
    - ` --no-commit `选项告诉 Git 此时无需自动生成和记录（合并）提交（即不论是否为快进模式，用户都要按照类似产生冲突时的做法来操作此次合并）
    > 注意，这种方式不会记录新增提交的来源是 aBranch，而是像在当前分支上直接提交的一样
##### 2.3.2 分支衍合（rebase）
> rebase 原意为 “重定基底”，在这里将用 rebase 方式整合分支的操作叫做 “分支衍合”
1. 用法
    - ` git rebase <targetRef> [sourceRef] `
        将` <sourceRef> `与` <targetRef> `不同的地方全都复制到` <targetRef> `后面（包括` <sourceRef> `指针）

        > 如果没有显式指明` <sourceRef> `，则默认为当前分支
        >
        > 与` git merge xxxBranch `类似，但可以理解为复制 xxxBranch 的所有提交记录到 master 分支，再把 xxxBranch 分支 “隐藏” 起来
        > 
        > ` git merge xxxBranch ` 相当于在 master 分支提交一个新的提交记录，并且这个记录同时指向 master 和 xxxBranch 分支最近的提交记录（@Q，待认证是否一定会产生一个提交记录，有没有办法不产生）
2. 一个例子
    从提交记录 C2 开始新建了一个分支叫 newBranch，并提交了一个 C3 提交记录，此时 master 分支也提交了一个 C4 提交记录（即 C3 和 C4 是平行的，分别属于 newBranch 和 master 分支），执行如下命令：
    ``` shell
    git checkout newBranch  # 切换到 newBranch 分支
    git rebase master       # 将 newBranch 分支的内容复制到 master 分支的后面（即在 C4 后面加一个 C3'，且 newBranch 分支指向 C3'）
    git checkout master     # 切换到 master 分支
    git merge newBranch     # 在 master 分支上合并 newBranch 分支（即使 master 分支指向 C3'）
    ```
    解释：回到两个分支最近的共同祖先（C2），根据当前分支（newBranch）后续的历次提交对象（这里只有一个 C3），生成一系列文件补丁，然后以基底分支（即 git rebase master 中指定的 master）最后一个提交对象（C4）为新的出发点，逐个应用之前准备好的补丁文件，最后会生成一个新的合并提交对象（C3'），从而改写 newBranch 的提交历史，使它成为 master 分支的直接下游，此时再合并就可以直接执行快进合并
3. 优势和适用场景
    - 优势
        衍合能产生一个更为整洁的提交历史。如果视察一个衍合过的分支的历史记录，看起来会更清楚：仿佛所有修改都是在一根线上先后进行的，尽管实际上它们原本是同时并行发生的。
    - 适用场景
        一般我们使用衍合的目的，是想要得到一个能在远程分支上干净应用的补丁。比如某些项目你不是维护者，但想帮点忙的话，最好用衍合：先在自己的一个分支里进行开发，当准备向主项目提交补丁的时候，根据最新的 origin/master 进行一次衍合操作然后再提交，这样维护者就不需要做任何整合工作（**实际上是把解决分支补丁同最新主干代码之间冲突的责任，化转为由提交补丁的人来解决**），只需根据你提供的仓库地址作一次快进合并，或者直接采纳你提交的补丁。

    > 请注意，合并结果中最后一次提交所指向的快照，无论是通过衍合，还是三方合并（即合并），都会得到相同的快照内容，只不过提交历史不同罢了。衍合是按照每行的修改次序重演一遍修改，而合并是把最终结果合在一起。
4. 注意事项
    一旦分支中的提交对象发布到公共仓库，就千万不要对该分支进行衍合操作。

    **真正出现问题的地方是：推送到远程仓库后再在本地 rebase，还使用` git push --force `将本次 rebase 的内容覆盖掉远程仓库的内容（主要问题出现在强制推送上，应竟可能少使用强制推送或谨慎使用）**

    建议：仅把衍合当成一种在推送之前清理提交历史的手段，而且仅仅衍合那些尚未公开的提交对象（即尚未推送到远程仓库的提交对象）
5. 交互式分支衍合
    ` git rebase -i <ref> `
    其中，
    ` <ref> `指的是目标提交，这个 **提交之后（不包括目标提交）** 的所有提交记录都会进入操作列表
    当 rebase UI界面打开时, 可以进行以下操作:
    - 调整提交记录的顺序
    - 删除你不想要的提交（通过切换 pick 的状态来完成，关闭就意味着你不想要这个提交记录）
    - 合并提交。它允许你把多个提交记录合并成一个
    > 具体应用可以参考 [重写历史](#重写历史)
##### 2.3.3 cherry-pick
` git cherry-pick <提交号>... `
可以用于将其他分支的提交复制到当前分支（其中，提交号可以为任意数量，使用空格隔开）
使用场景：在功能分支或 bug 分支中提交了很多提交，但只需要将最后一个提交合并到主分支
> 使用` git rebase -i xxx `配合使用` git rebase xxBranch `也能达到同样的效果。区别在于，使用 cherry-pick 看不出来提交是从其他地方来的，就像是在 master 分支上使用 git commit 提交的，而使用 rebase 可以知道该提交记录来源于其他地方
>
> cherry-pick 成功的基础：每次提交保存的是快照而不是增量变化
#### 2.4 删除分支
- 删除前
    - git branch --merged
        查看哪些分支已被并入当前分支（译注：也就是说哪些分支是当前分支的直接上游）。其中，结果显示的分当前分支都可以删除，因为已经把它们所包含的工作整合到了当前分支，删掉也不会损失什么。
    - git branch --no-merged
        查看哪些分支尚未被并入当前分支。由于这些分支中还包含着尚未合并进来的工作成果，所以简单地用` git branch -d xxxBranch `删除该分支会提示错误，因为那样做会丢失数据（但可是将` -d `替换为` -D `来强制删除）。
- 删除
    ` git branch -d branchName `
    删除 branchName 分支，如果该分支里的提交没有完全合并则会提示使用` -D `来删除（不能在当前分支上删除当前分支）
#### 2.5 Romote branch 远程分支
定义：远程分支（remote branch）是对远程仓库中的分支的索引。它们是一些无法移动的本地分支；只有在 Git 进行网络交互时才会更新。远程分支就像是书签，提醒着你上次连接远程仓库时上面各分支的位置。 
#### 2.6 同步远程分支到本地
` git fetch origin `
> fetch 为 “抓取” 的意思
同步远程服务器上的数据到本地（注意，此时只是更新本地仓库的远程分支，尚未合并）
#### 2.7 跟踪远程分支
从远程分支 checkout 出来的本地分支，称为 跟踪分支 (tracking branch)。跟踪分支是一种和某个远程分支有直接联系的本地分支。在跟踪分支里输入` git push `，Git 会自行推断应该向哪个服务器的哪个分支推送数据。同样，在这些分支里运行` git pull `会获取所有远程索引，并把它们的数据都合并到本地分支中来。
在克隆仓库时，Git 通常会自动创建一个名为 master 的分支来跟踪 origin/master。这正是 git push 和 git pull 一开始就能正常工作的原因。

使用：
- ` git checkout -b <branchNameA> <remoteRepName>/<branchNameB> `
创建一个 branchNameA 分支，该分支跟踪远程仓库 remoteRepName 的 branchNameB 分支

- ` git branch -u <remoteRepName>/<branchNameB> [branchNameA] `
设置本地仓库的分支 branchNameA 跟踪远程仓库 remoteRepName 的 branchNameB 分支。如果省略 branchNameA，则默认为当前分支
#### 2.8 删除远程分支
` git push [远程名] :[分支名] `

### 3. Stashing 储藏
#### 3.1 作用
Stashing 可以获取工作目录的中间状态，包括修改过的被追踪的文件（即修改但未暂存的文件）和暂存的变更（即已经使用 git add 添加了的文件），并将它保存到一个未完结变更的堆栈中，随时可以重新应用。
#### 3.2 使用场景
当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点。
#### 3.3 命令
- ` git stash `储藏当前的变化
    会被储藏的内容：
    - 经过修改的工作区被追踪文件（即` Changes not staged for commit: `下的文件）
        > 不包括 **新建** 但尚未使用 git add 的文件，即` untracked files: `下的文件，这些文件在执行完 stash 之后依然在原位置。如果需要储藏这些文件可以使用` git stash push [-u|--include-untraked] `，使用` -a|--all `的话可以把 .ignore 里面的文件都储藏起来
    - 暂存区的所有内容（即` Changes to be committed: `下的文件）
    > 建议：使用` git stash push -m "message" `来储藏内容并带上一些提示信息，利于辨别储藏记录
- ` git stash list `查看现有的储藏记录
@todo 放恢复的内容
- ` git stash drop [stashName] `移除栈上的储藏记录
    apply 只是应用储藏的工作，但不会自动删除栈上的记录，需要手工删除。不指明 stashName 的话将删除最近的一个储藏，或者使用` git stash pop `在应用储藏的同时移除栈上的记录
    @Q，这个命令是否可以完全恢复
- ` git stash branch branchName [stashName]`从储藏中创建分支
    使用场景之一：储藏之后可能会在该分支修改了很多内容导致直接在本分支应用储藏会发生冲突。
    此时可以使用该命令创建一个新的分支，检出储藏的内容。
    > 注意，这种方式可以完全恢复储藏前的状态而且会自动删除栈上的该储藏记录
    @Q，会不会自动提交一次
- 较少使用
    - ` git stash show [stashName] `查看指定 stash 的具体内容
    - ` git stash apply [stashName] `应用一个储藏（不完全恢复）
        不指明 stashName 的话将默认应用最近的一个储藏，stashName 可以在执行` git stash list `后查看，格式大概为` stash@{num} `
        > 注意：这种方式不能完全恢复到储藏前的状态，只会重新应用对文件的变更，不会重新暂存储藏前已被暂存的文件，@Q，貌似可以的，不知道 --index 的真实意思
    - ` git stash apply --index [stashName] `应用一个储藏（完全恢复）
        不指明 stashName 的话将默认应用最近的一个储藏，使用` --index `可以 **完全恢复** 到暂存前的状态
    - ` git stash show -p [stachName] | git apply -R `应用储藏后又需要取消应用储藏(Un-applying a Stash)
        在某些情况下，你可能想应用储藏的修改，在进行了一些其他的修改后，又要取消之前所应用储藏的修改。Git没有提供类似于 stash unapply 的命令，但是可以通过取消该储藏的补丁达到同样的效果
        不指明 stashName 的话将默认取消应用的最近的储藏
        @Q，需要明确确定 stashName 的意义，如果没有应用该 stashName 的储藏要怎么取消该储藏？能不能不写 stashName


### 4. Tag 标签
#### 4.1 分类
- 轻量级的（lightweight）

    ` git tag tagName `

    轻量级标签就像是个不会变化的分支（注意，分支其实就是一个指针，这种标签可以理解为），实际上它就是个指向特定提交对象的引用。
- 含附注的（annotated）

    ` git tag -a tagName -m '附注内容' `（其中，` -a `取 annotated 的首字母）

    实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及 **标签说明** ，标签本身也允许使用 GNU Privacy Guard (GPG) 来签署或验证。

> 建议使用含附注型的标签，以便保留相关信息；当然，如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签也没问题。
#### 4.2 查看标签信息
` git show tagName `显示相应标签的版本信息
#### 4.3 推送标签
` git push origin [tagName] `推送指定的 tag 到远程仓库 origin
` git push origin --tags `推送本地新增的 tags 到远程仓库 origin

### 5. 查看提交
#### 5.1 提交记录的移动
- ` git checkout HEAD^ `HEAD 向上移动一个提交记录（可以连续使用，例如，` git checkout HEAD^^^`）
    
    > ` ^ ` 后面可以加数字，构成` ^[num] `，但和` ~[num] `意义不同（num 从 1 开始，不是从 0）
    >
    > 主要应用在一个提交记录有多个父提交记录（例如，使用 merge 之后就会出现这种情况），指定移动到哪个父提交记录的场景，或者使用` git diff `对比不同提交记录中的同一记录
- ` git checkout HEAD~[num] `HEAD 向上移动 num 个提交记录，不加` [num] `即 num=1

综合应用例子：
` git checkout HEAD~^2~2` 1.向上移动一个提交记录 2.选第二个父提交记录向上 3.向上移动两个提交记录
#### 5.2 查看分支间差异的提交记录
- ` git log master..aBranch `两点语法
    可以用来查看你的 aBranch 分支上哪些没有被提交到主分支 master，那么你就可以使用 master..aBranch 来让 Git 显示这些提交的日志——这句话的意思是 “所有可从 aBranch 分支中获得而不能从 master 分支中获得的提交”（即 aBranch 有而 master 没有）
    > 可以将 master 和 aBranch 调换位置，查看 master 有而 aBranch 没有的
- ` git log master...aBranch `三点语法
    可以可以查看两个引用中的一个包含但又不被两者同时包含的分支，添加参数` --left-right `可视化更加好，即` git log --left-right master...aBranch `
#### 5.3 显示最近的一个标签
` git describe <ref> `
显示距离指定 **引用**（分支、提交）最近的一个 tag
其中：
` <ref> `可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（HEAD）(当` <ref> `提交记录上刚好有某个标签时，则只输出标签名称)
输出格式：
` <tag>_<numCommits>_g<hash> `
tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。
#### 5.4 显示文件间差异
- ` git diff fileName `
    对比当前文件和暂存区域中的文件的差异
- ` git diff --cached `或` git diff --staged `
    对比暂存区中的文件和上次提交时的快照之间的差异

### 6. 撤销变更
#### 6.1 git reset
` git reset HEAD^`之后，会移动到上一个提交 C1（本地代码库不知道最近的一个提交 C2），但是提交 C2 的修改还在，现在变成了未加入暂存区的状态
> 对远程分支无效，要使用` git revert `
#### 6.2 git revert
` gir revert HEAD `会创建一个提交，该提交是前一个提交的逆过程（就是用来撤销前一个提交，回退到上上个提交）
#### 注意区别
> ` git reset `后面接的是 “**要撤销到的那个节点**”（注意，**到**），例如，当前提交为 C1->C2->C3->C4(当前) ，要撤销到 C2 提交时的状态，那就使用` git reset C2 `来达到目的；
> 
> 而` git revert `后面接的是 “**要撤销的那个节点**”（注意，没有 **到**），例如，当前提交为 C1->C2->C3->C4(当前) ，要撤销到 C2 提交时的状态（这里是 **C2**），那就使用` git revert C3 `（这里是 **C3**）来达到目的；
>
> 根因：reset 是指 “重新设置到某个提交”，revert 是指 “撤销某个提交”

### 7. 重写历史
#### 7.1 改变最近一次提交
> 使用场景：提交之后，发现提交漏了文件或者提交信息错误，这时候又不想修改完再提交一次，就可以使用下面命令修改原来的提交（类似于覆盖原来的提交，但 git 再也查不出覆盖前任何信息）
1. 修改最近一次提交的说明
    执行` git commit --amend `，在弹出的文本编辑器修改提交说明，保存即覆盖原来的提交说明
2. 修改最近一次提交的快照（即增、删、改快照中的文件）
    先增、删、改文件，再添加到暂存区，再执行` git commit --amend `，进入编辑器后如果不需要修改的话直接退出即可
> 注意：使用该命令必须小心，因为修正会改变提交的 SHA-1 值，类似于一次 rebase，所以**不要在你最近一次提交被推送后还去修正它**

    @Q，待测试，未确定。
    - 场景一：漏了文件 readme.md（如果连提交信息都要修改的话将` --no-edit `修改为场景二类似即可）
        ``` shell
        $ git commit -m 'update'
        $ git add readme.md
        $ git commit --amend --no-edit
        ```
    - 场景二：提交信息错误
        ``` shell
        $ git commit -m 'updata'
        $ git commit --amend -m 'update'
        ```
    - 场景三：提交多了文件
        ``` shell
        $ git commit -m "update"
        $ git reset HEAD^ fileName
        $ git commit --amend --no-edit
        ```
    > 貌似直接使用` git commit --amend `会弹窗输入提交信息
#### 7.2 修改多个提交说明
使用` git rebase -i <ref> `进行交互式 rebase（根据提示操作，主要是 edit）
#### 7.3 重排提交
重新排列提交记录的顺序（在交互式 rebase 中根据提示操作，主要是 edit）
#### 7.4 压制(Squashing)提交
将多个提交合并为一个提交（在交互式 rebase 中根据提示操作，主要是 sqush）
#### 7.5 拆分提交
将提交拆分为多个提交（在交互式 rebase 中根据提示操作，主要是 edit）
#### 7.6 批量修改提交
可以使用` filter-branch `配合脚本修改大量的提交
- 例子
    全局性地修改电子邮件地址或者将一个文件从所有提交中删除（把记录着密码的配置文件删除等）。该命令会大面积地修改历史，所以你很有可能不该去用它，除非你的项目尚未公开，没有其他人在你准备修改的提交的基础上工作。
- 使用场景
    1. 从所有提交中删除一个文件
        ` git filter-branch --tree-filter 'rm -f xxx.txt' HEAD `
        其中，
        --tree-filter 选项会在每次检出项目时先执行指定的命令然后重新提交结果
        rm -f xxx.txt 即脚本
        > 建议：一个比较好的办法是在一个测试分支上做这些然后在你确定产物真的是你所要的之后，再 hard-reset 你的主分支。要在你所有的分支上运行 filter-branch 的话，你可以传递一个 --all 给命令。
    2. 更多用法参考帮助手册

### 8. 远程仓库
#### 8.1 概念
- 本地仓库与远程仓库

    本地仓库就是在本地创建的仓库，远程仓库一般托管在 Github, GitLab 上面
- 远程分支

    远程分支 **特指在本地仓库中会跟踪指定远程仓库的分支**，只要本地仓库和远程仓库建立了联系就会自动生成，**不是 “远程仓库中的分支” 的意思**
#### 8.2 远程分支
远程分支反映了远程仓库在你最后一次与它通信时的状态。在使用` git clone xxxx `之后，会在本地仓库自动创建一个远程分支，默认名字为 origin/master

远程分支的命名规范` <remote name>/<branch name> `，一般为` origin/master `
> 注意：即使切换到远程分支，在远程分支上提交内容也不会更新，**远程分支只会在远程仓库中相应的分支更新了以后才会更新**。
#### 8.3 从远程仓库获取数据
##### git fetch
作用：将本地仓库的远程分支更新到和远程仓库的同名分支相同的状态（可以理解为单纯地下载，不会自动合并到当前工作分支）

具体：
1. 从远程仓库下载本地仓库中缺失的提交记录
2. 更新本地的远程分支指针(如 origin/master)

进阶: ` git fetch <remote> <place>`
例子：
git fetch origin foo
Git 会到远程仓库的 foo 分支上，然后获取所有本地不存在的提交，放到本地的 o/foo 上。

进阶：` git fetch <remote> <source>:<destination> `（基本不会使用）
source 是远程仓库的任意一个 ref
destination 是本地仓库的分支（**但不能是当前分支**，如果不存在会在本地仓库自动新建一个）

如果省略 source，即` git fetch origin :bar `，会在本地仓库新建一个 bar 分支（如果本地仓库中存在同名则不允许新建）

##### 获取了数据之后，就可以使用下面的命令来将远程分支的内容合并到本地：
- ` git cherry-pick origin/master `
- ` git rebase origin/master `
- ` git merge origin/master `

##### git pull（简便操作）
git pull = git fetch + git merge

##### git pull --rebase
git pull --rabase = git fetch + git rebase
#### 8.4 向远程仓库传输数据
> git push 不带任何参数时的行为与 Git 的一个名为 push.default 的配置有关。它的默认值取决于你正使用的 Git 的版本，但是在教程中我们使用的是 upstream。 这没什么太大的影响，但是在你的项目中进行推送之前，最好检查一下这个配置。

具体：
1. 将本地仓库的提交记录上传到远程仓库
2. 更新远程仓库的同名分支
2. 更新本地的远程分支指针

##### git push <remote-name> <branch-name>
例子：
git push origin master 把这个命令翻译过来就是：
切到本地仓库中的 “master” 分支，获取所有的提交，再到远程仓库 “origin” 中找到 “master” 分支，将远程仓库中没有的提交记录都添加上去，搞定之后告诉我。
> 即，把 branch-name 分支的内容提交到 remote-name 远程仓库的同名分支

进阶：` git push <remote> <source>:<destination> `
remote 基本上只能是 origin
source 是本地仓库的任意一个 ref
destination 是远程仓库中的分支（如果不存在会自动新建）
例子：
1. git push origin foo^:master
    会将 foo 的上一个提交记录推送到 origin/master
2. git push origin master:newBranch
    会将 master 的提交记录推送到 origin/newBranch（如果 newBranch 不存在则先在远程仓库新建一个）

如果省略 source，即` git push origin :foo `，会删除远程仓库的 foo 分支（如果远程仓库不存在则不会删除）
#### 8.5 添加远程仓库
` git remote add [remoteRepName] [url] `

### 9. 其他
#### 9.1 git 的配置
> 可以使用` git config --list `来查看配置信息
git 存在三个配置文件：
1. /etc/gitconfig
    对系统中所有用户都普遍适用的配置，对应` git config --system `
2. ~/.gitconfig
    用户目录下的配置文件只适用于该用户，对应` git config --global `
3. .git/config
    当前项目的 git 目录中的配置文件仅仅针对当前项目有效，对应` git config [--local] `
> git config 的意义：最简单的是明确提交时的作者和邮箱
#### 将本地目录初始化为 git 仓库
新建一个目录，在目录中执行` git init `即可
> 不直观的直观表现：当前目录中多了一个隐藏的 .git 目录（没有完全把握切勿修改该目录内文件）
#### 9.2 git 的帮助
1. ` git help <verb> `
2. ` git <verb> --help `
3. ` man git-<verb> `
#### 9.3 .gitignore 文件
作用：忽略跟踪 .getignore 文件中指定的文件或目录（即不会暂存和提交这些文件）

可以使用标准的` glob 模式 `匹配，所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。
- ` * `（星号）匹配零个或多个任意字符；
- ` [abc] `匹配任何一个列在方括号中的字符（要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；
- ` ? `（问号）只匹配一个任意字符；
- ` [0-9] `如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）
- 匹配模式最后跟` / `（斜杠）表示目录
- 要忽略指定模式以外的文件或目录，可以在模式前加上` ! `（感叹号）取反
#### 9.4 git commit -a
Git 会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过` git add `步骤
> 可以提交新添加的文件？应该不行。是否需要输入提交说明？（@Q）
#### 9.5 git rm --cached <fileName>
把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中（即从跟踪清单中删除）
> 类似于将文件加入到 .gitignore ？（@Q）


## Github
### GitHub 的 SSH Keys 设置
参考 [官网](https://help.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)
### 在 Github 中精准搜索项目
- `in:name` 在名字中搜索
- `in:readme` 在 readme 中搜索
- `in:description` 在描述中搜索（**常用**）
- `starts:>200` 搜索超过 200 starts
- `forks:>200` 搜索超过 200 forks
- `language:java` 指定语言为 java
- `pushed:>2019-09-03` push 时间大于 2019-09-03