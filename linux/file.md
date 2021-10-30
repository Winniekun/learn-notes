## 序

Linux中最优秀的一个地方在于他的多用户多任务环境。提供多用户多环境很重要的一个方面就是文件权限的管理

在Linux中一般将文件的存取身份分为三大类：owner/group/others，同时三种身份又同时拥有read/write/execute等权限

### owner、group、others阐述

记录下三者的阐述，使用树状结构来看三者关系会更加明朗

#### owner

因为Linux是提供用户多任务环境，因此可能同时会有多个用户访问同一个主机进行工作，考虑到每个用户的隐私以及个人自定义的环境，由此引出了owner的概念

#### group

在共同开发的过程中，涉及到协同开发，同一组的人员能够共同编辑一些文件，同时为了避免不必要的麻烦，不同的组之间不能互相查看修改这些内容。

所以，group的设置是使得某些文件的受众范围相比owner更大些

#### others

可以简答的理解成其他部门的人员

## Linux文件属性

![img](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/1634888426826-5a0cf6d4-858d-460f-a62e-c8b57e2abe2c.png)

顺序为 owner group others，同时不同的组的文件权限保持不变，统一为r、w、x

**举例：**

```bash
 drwxr-xr--   1 test1    testgroup    5238 Jun 19 10:25 groups
```

1. 文件拥有者test1[rwx]可以在本目录中进行任何工作

1. testgroup这个群组[r-x]的帐号，例如test2, test3亦可以进入本目录进行工作，但是不能 在本目录下进行写入的动作
2. other的权限中[r--]虽然有r ，但是由于没有x的权限，因此others的使用者，并不能进入此目录! 

### 文件权限的重要性

- 系统保护的功能

  举个简单例子，在你的系统中，关于系统服务的文件通常只有 root 才能读写或执行，
  例如 `/etc/shadow` 这个账户管理的文件，这个文件的是个字符都是横线，不能读写执行，
  但是 root 不受限制

- 团队开发软件或数据共享的功能

  就是多人协作的时候，希望每个人都可以使用某一些目录下的文件，而其他人不开放。
  比如 testgroup 团队有三个人 t1、t2、t3 ，那么就可以将团队所需的文件权限设置为 `-rwxrws---`
  该组内的都可读写与执行（等等，这里怎么是 s? 后续会讲解）

- 未将权限设置妥当的危害

  很简单，比如只有 root 才能做的开关机，新增、或删除用户等等的指令，那么随意人都可以用的话，
  就乱套了

### 如何改变文件属性与权限 

通过上述对Linux文件属性的阐述，接下来需要就需要去了解如何去修改文件的权限了，以供我们的一些需求

1. chgrp（**change group**）

   >  改变文件所属群组

   譬如我们将`initial.cfg`的`group`改为`group1`

   ![chgrp的使用](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211030214432455.png)

   

1. chown（**changer owner**）

   > 改变文件拥有者

   譬如我们将`initial.cfg`文件的`owner`改为wkk，然后在重新将其的`owner`和`group`重新改回`root`

   ![image-20211030214932847](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211030214932847.png)

1. chmod

   > 文件权限改变

   有两种形式改变文件权限：`数字类型`、`符号类型`

   **数字类型：**

   - r：4
   - w：2
   - x：1

   每种身份(owner/group/others)各自的权限(r/w/x)的累加求和，譬如：[-rwxrwx---]

   - owner: r + w + x = 4 + 2 + 1 = 7
   - group: r + w + x = 4 + 2 + 1 = 7
   - others: - - - = 0 + 0 + 0 = 0

   ![chmod举例](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211030220407753.png)

   常用的数字组合：

   1. 755 （-rwxr-xr-x）

      > 文件自己可读可写可执行，并且不允许别人修改

   2. 740 （-rwxr-----）

      > 文件除了自己之外，不希望别人看到

   **符号类型：**

   ![image-20211030221232883](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211030221232883.png)

   譬如我们希望`chmodtest.cfg`仅对`owner`具有`r、w、x`的权限，`group、owner`具有`r、x`权限

   ![chmod](https://cdn.jsdelivr.net/gh/Winniekun/cloudImg@master/uPic/image-20211030221601727.png)

