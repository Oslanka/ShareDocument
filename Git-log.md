# 深入浅出解析Git-log，并实战统计项目（后端篇）

```bash
# Author: 竹叶青 
# Date:  2021-04-23
# Email: oslankachina@gmail.com
```

# Git-log

## 前言

Git-log - 查看git 提交日志，本篇会全面解析git-log，如果你是个管理者，那么你一定在乎手下做了什么，提了什么东西，是否有“佛主保佑”，也可帮你code review，还能帮你 blame someOne  ，你想blame吗？你想快速定位你的代码吗？那就看看我们的git log 到底能做点啥吧，Go On。

## 概要

```bash
$ git log [<options>] [<revision range>] [[--] <path>…]
```

- 本文中多会用到常用以下两种命令做demo，一种普通log，一种单行显示log

  ```bash
  # 普通log
  $ git log
  # 单行显示
  $ git log --oneline
  ```


## 选项options

- --follow:会追溯历史，包括重命名前的记录

  ```bash
  $ git log -p gradle.properties
  # 不带 follow参数的情况下，git把名为gradle.properties的文件全部提交记录找出来
  $ git log -p --follow gradle.properties
  # 带follow参数的情况下，git会把当前以及重命名前的文件提交记录找出来
  ```

- --no-decorate:不显示的所有提交的引用名称

  ```bash
  $ git log --no-decorate
  ```
  
  ```bash
  commit hash
  Author: caining <demo@gmail.cn>
  Date:  Tue Apr 20 11:10:53 2021 +0800
  develop2
  ```
  
- --decorate[=short|full|auto|no]:显示的所有提交的引用名称

  - **名词解释：Git Reference**
    - Git Reference简写为refs
    - 本地分支的Reference格式：refs/heads/<local_branch_name>如refs/heads/master，在保证唯一的情况下可以简写为master；
    - 远程追踪分支的Reference格式：refs/remotes/<remote_repository>/<remote_branch_name>：如refs/remotes/origin/master，在保证唯一的情况下可以简写为origin/master；
    - tag的Reference格式：refs/tags/<tag_name>：如refs/tags/v1.0.1，在保证唯一的情况下可以简写为v1.0.1
    - 特殊refs-HEAD，指向当前本地分支的当前commit状态
    - 特殊refs-FETCH_HEAD，指向当前本地分支在最近一次fetch操作时得到的commit状态
    - 特殊refs-ORIG_HEAD，指向任何merge或rebase之前的刚刚检出时的commit状态
  ```bash
  $ git log --decorate=full
  # 结果：HEAD -> refs/heads/develop , refs/remotes/origin/develo
  $ git log --decorate=short
  # 结果：HEAD -> develop, origin/develop
  ```

- --source:均显示提交分支

  ```bash
  # 每个提交均显示提交分支
  $ git log --oneline --source
  # 6256aa140       HEAD (HEAD -> develop, origin/develop) message
  ```

- --log-size:是该提交消息的长度（以字节为单位）

  ```bash
  $ git log --oneline --log-size
  
  # commit 6256aa140f910d81eb71a7651aad1d6bc5128da0 (HEAD -> develop, origin/develop)
  # log size 107
  # Author: caining <oslankachina@gmail.com>
  # Date:  Thu Apr 22 09:59:29 2021 +0800
  # message
  ```

- -L<start>,<end>:<file>:查看某个文件某几行范围内的修改记录

- -L:<funcname>:<file>:追踪一个函数的变更历史

  - 但是，为了使其工作，Git需要能够确定哪些行是函数声明。它有一个默认的正则表达式，它假定以字母，下划线或美元符号（无前导空格）开头的任何行都是函数声明的开头，并使用-L：<funcname>：<file>语法将搜索也与您的模式匹配的函数声明，直到下一个函数声明或文件末尾。
  
  - 在某些语言中，这种启发式是不合适的。例如，如果函数或方法声明嵌套在类的内部并因此缩进，则不会在这些声明中使用。为此，您需要定义一个自定义函数头正则表达式。 .gitattributes部分“定义自定义的块头”描述了如何执行此操作。首先，您将在项目的顶层创建一个.gitattributes文件，其中包含以下内容：
  
  - ```swift
    *.swift diff=swift
    ```
  
- ```bash
  $ git log -L 1,1:README.md
  $ git log -L :MainActivity:MainActivity.java
  ```
  
  <img src="https://www.hualigs.cn/image/60829973b2372.jpg" alt="image-20210423175510685" style="zoom:50%;" />


## 重点-筛选

- 请注意，这些是在提交顺序和格式设置选项（例如--reverse）之前应用的。显示提交日志.以下 a b c 均为提交节点hash 值 commit时间 a< b < c )

  ```bash
  # 普通log
  $ git log
  # 单行显示
  $ git log --oneline
  ```

  ```bash
  # 列出a到(画重点：理解^ 可以理解为from  前面大 后面小，..可以理解为to 前面小后面大)
  $ git log a..b
  $ git log b ^a
  ```

  ```bash
  #列出所有可以从a 到 b而不包含c之前的提交
  $ git log a b ^c
  ```

- 数量筛选

  ```bash
  $ git log -3
  $ git log -n3
  $ git log --max-count=3
  # -<number> 
  # -n <number>
  # --max-count=<number>
  # --skip=<number>
  $ git log --skip=3 #跳过3条记录
  ```

- 时间筛选

  ```bash
  $ git log --since='2021-04-02 00:00:00'
  # --since=<date>
  # --after=<date>
  $ git log --until='2021-04-02 00:00:00'
  # --until=<date>
  # --before=<date>
  # 2周内的提交
  $ git log --since=2.weeks = $ git log --since="2 weeks ago"
  # 2月内的提交
  $ git log --since=2.months = $ git log --since="2 months ago"
  # 1年内的提交
  $ git log --since=1.year = $ git log --since="1 year ago"
  ```

- 作者筛选

  ```bash
  $ git log --author='caining'
  # --author=<pattern>
  # --committer=<pattern>
  # 注意这里是正则，模糊匹配，
  # 如果是多个人人一起查按下面
  $ git log -author="name1\|name2\|name3"
  ```

- 提交描述筛选

  ```bash
  $ git log --grep='bug fix'
  $ git log -i --grep="message1\|message2"
  # --grep=<pattern>
  # 重点1： --all-match 可以理解为sql-and与 --grep 至少有一个匹配的提交 例如与-author混用时
  $ git log --author='caining' --all-match --grep='bug fix'
  # 重点2： --invert-grep 用法同上，可以理解为sql not --grep 至少有一个不匹配的提交 例如与-author混用时
  $ git log --author='caining' --invert-grep --grep='bug fix'
  #上句理解： 为caining 提交的 但是 描述里不包含 bug fix的 提交内容
  ```

- 代码内容搜索 -S，注意这个搜索可能很耗时。

  ```bash
  $ git log -S hello
  ```

<img src="https://www.hualigs.cn/image/60827bf539c37.jpg" alt="image-20210423154604846" style="zoom:50%;" />

- 多条件混合筛选

  ```bash
  $ git log --since=1.year --author='caining' --grep='获取' -3
  ```

  

<img src="https://www.hualigs.cn/image/6082920c4a635.jpg" alt="image-20210423172328921" style="zoom:50%;" />

## format与样式

- --pretty[=<format>

- --date=<format>：`--date=relative``--date=local``--date=default-local`.`--date=iso` `--date=iso8601``--date=rfc``--date=short``--date=rfc2822`

  日期格式化

- --parents

  显示提交的父母节点

- --children

  显示提交的孩子节点

- --graph 

  --graph like this<img src="https://www.hualigs.cn/image/60910d1a51d38.jpg" alt="image-20210504160756705" style="zoom: 50%;" />

- git log --pretty=format   已存在样式包含 `oneline`,`short`, `medium`, `full`, `fuller`, `reference`, `email`, `raw`

  ```bash
  $ git log --pretty=oneline
  # 样式:
  # <hash> <title line>
  ```

  ```bash
  $ git log --pretty=short
  # 样式:
  # commit <hash>
  # Author: <author>
  # <title line>
  ```

  ```bash
  $ git log --pretty=medium
  # 样式:
  # commit <hash>
  # Author: <author>
  # Date:   <author date>
  # <title line>
  # <full commit message>
  ```

  ```bash
  $ git log --pretty=full
  # 样式:
  # commit <hash>
  # Author: <author>
  # Commit: <committer>
  # <title line>
  # <full commit message>
  ```

  ```bash
  $ git log --pretty=fuller
  # 样式:
  # commit <hash>
  # Author:     <author>
  # AuthorDate: <author date>
  # Commit:     <committer>
  # CommitDate: <committer date>
  # <title line>
  # <full commit message>
  ```

- 自定义git log --graph --pretty=format:''

  ```bash
  #%Cred	切换至红色
  #%Cgreen	切换至绿色
  #%Cblue	切换至蓝色
  #%C(yellow)切换至黄色
  #%Creset 重设颜色
  $ git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
  ```

<img src="https://www.hualigs.cn/image/60910d7686d06.jpg" alt="image-20210501171014592" style="zoom:50%;" />

- format支持的类型如下，以%开头：

  ```bash
  # %H:完整哈希字串
  # %h:简短哈希字串
  # %T:树对象（tree）的完整哈希字串
  # %t:树对象的简短哈希字串
  # %P:父对象（parent）的完整哈希字串
  # %p:父对象的简短哈希字串
  # %an:作者（author）的名字
  # %aN:作者（author）的名字 (respecting .mailmap, see [git-shortlog[1\]](https://git-scm.com/docs/git-shortlog) or [git-blame[1\]](https://git-scm.com/docs/git-blame))
  # %ae:作者电子邮件
  # %aE:作者电子邮件 (respecting .mailmap, see [git-shortlog[1\]](https://git-scm.com/docs/git-shortlog) or [git-blame[1\]](https://git-scm.com/docs/git-blame))
  # %al:作者电子邮件的本地部分（@号之前的部分）
  # %aL:author local-part (see *%al*) respecting .mailmap, see [git-shortlog[1\]](https://git-scm.com/docs/git-shortlog) or [git-blame[1\]](https://git-scm.com/docs/git-blame))
  # %ad:作者日期 (format respects --date= option)
  # %aD:author date, RFC2822 style
  # %ar:作者日期，相对
  # %at:作者日期，UNIX时间戳
  # %ai:author date, ISO 8601-like format -作者日期，类似IS0 8601的格式
  # %aI:author date, strict ISO 8601 format - 作者日期，严格的lS0 8601格式
  # %as:作者日期，短格式（YYYY-MM-DD）
  # %cn:提交者名称
  # %cN:committer name (respecting .mailmap, see [git-shortlog[1\]](https://git-scm.com/docs/git-shortlog) or [git-blame[1\]](https://git-scm.com/docs/git-blame))
  # %ce:提交者电子邮件
  # %cE:committer email 
  # %cl:committer email local-part 
  # %cL:committer local-part 
  # %cd:committer date (format respects --date= option)
  # %cD:committer date, RFC2822 style
  # %cr:committer date, relative
  # %ct:committer date, UNIX timestamp
  # %ci:committer date, ISO 8601-like format
  # %cI:committer date, strict ISO 8601 format
  # %cs:committer date, short format (`YYYY-MM-DD`)
  # %s:message
  
  $ git log  --graph --pretty=format:'提交:-:%Cred%h%Creset;%Cgreen日期:-:%ar%Creset;%Cblue日期at:-:%at%Creset;%C(yellow)日期ai:-:%ai%Creset;%Cred日期ad:-:%ad%Creset;%Cblue作者an:-:%an%Creset;%C(yellow)作者aN:-:%aN%Creset;%Cgreen邮箱ce:-:%ce%Cresete:-:%e;sssss:-:%Cred%s%Creset;SSSS:-:%S;f-%f%n'
  ```

<img src="https://www.hualigs.cn/image/60910eabb9a96.jpg" alt="image-20210501160914927" style="zoom:50%;" />

## git shortlog--统计

```bash
# 默认以贡献者分组进行输出
$ git shortlog

# 列出提交者代码贡献数量, 打印作者和贡献数量
$ git shortlog -sn==git shortlog -s -n
$ git log --pretty='%an' | sort | uniq -c | sort -k1 -n -r | head -n 10
#-s 参数省略每次 commit 的注释，仅仅返回一个简单的统计。
#-n 参数按照 commit 数量从多到少的顺利对用户进行排序

# 以提交贡献数量排序并打印出message
$ git shortlog -n

# 采用邮箱格式化的方式进行查看贡献度
$ git shortlog -e
```

```bash
$ git shortlog -sn==git shortlog -s -n
# &&
$ git log --pretty='%an' | sort | uniq -c | sort -k1 -n -r | head -n 10
```

<img src="https://www.hualigs.cn/image/6091082cda152.jpg" alt="image-20210501162714228" style="zoom:50%;" />

## git blame--查提交

```bash
# 查看 README.md 文件的修改历史记录，包括时间、作者以及内容
git blame README.md

# 查看谁改动了 README.md 文件的 11行-12行
git blame -L 11,12 README.md
git blame -L 11 README.md   # 查看第11行以后

# 显示完整的 hash 值
git blame -l README.md

# 显示修改的行数
git blame -n README.md

# 显示作者邮箱
git blame -e README.md

# 对参数进行一个组合查询
git blame -enl -L 11 README.md
```



# 实战项目

- 根据需求导出所需要的提交日志内容to本地文件

  ```bash
  $ git log --date=iso --pretty=format:’"%h","%an","%ad","%s","%ce"’ >> ~/Desktop/commit.csv
  $ git log  --date=iso --pretty=format:’"%h","%ar","%at","%ai","%ad","%an","%aN","%ce","%e","%s","%S","%f"’ >> ~/Desktop/commit.csv
  $ git log --date=iso --pretty=format:’"项目-demo","%H","%h","%an","%ae","%at","%ad","%s"’ >> ~/Desktop/项目-demo.csv
  #%H:完整哈希%h:简短哈希%an:作者%ae:邮件 %at:作者日期，UNIX时间戳"%ad","%s"
  # 我们根据 git shortlog -sn，统计出排名，然后查看 Yangkai.Shen，yangkai.shen的代码行数
  $ git log --author="Yangkai.Shen\|yangkai.shen" --pretty=tformat: --numstat | awk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END { printf "增加的行数:%s 删除的行数:%s 总行数: %s\n",add,subs,loc }'>> ~/Desktop/项目-demo_code.csv
  # 增加的行数:133582 删除的行数:77429 总行数: 56153
  
  git log --author="Yangkai.Shen\|yangkai.shen" --pretty=tformat: --numstat | awk '{ add += $1 ; subs += $2 ; loc += $1 - $2 } END { printf "%s,%s,%s,%s\n",add,subs,loc,"Yangkai.Shen" }'>> ~/Desktop/项目-demo_code.csv
  #658  Yangkai.Shen
  #    57  yangkai.shen
  #     4  fxbin
  #     3  EchoCow
  #     2  chenqi
  #    1  yaodehuang
  #     1  76peter
  #     1  yidasanqian
  #     1  UndefinedDiary
  #     1  geek
  #     1  taffier
  #     1  yanshaoshuai
  ```

<img src="https://www.hualigs.cn/image/609104b536c56.jpg" alt="image-20210502193902849" style="zoom:50%;" />

- 利用Python 脚本循环读取作者，可以获取各个作者代码数量，组装为json，并导出，关键代码如下：

```python
if __name__ == '__main__':
    p = subprocess.Popen("git log --pretty='%an' | sort | uniq -c | sort -k1 -n -r | head -n 10", shell=True,
                         cwd='/Users/caining/Documents/demo/spring-boot-demo', stdout=subprocess.PIPE)
    out, err = p.communicate()
    value = str(out, encoding="utf-8")
    arr = value.split("\n")
    subprocess.Popen('printf [ >> ~/Desktop/项目-demo_code1.csv', shell=True,
                     cwd='/Users/caining/Documents/demo/spring-boot-demo')
    for a in arr:
        if len(a.split(" ")) >= 2:
            name = a.split(" ")[len(a.split(" ")) - 1]
            author = name
            subprocess.Popen(
                'printf {\\"name\\":\\"' + author + '\\",\\"score\\":\\" >> ~/Desktop/项目-demo_code1.csv', shell=True,
                cwd='/Users/caining/Documents/demo/spring-boot-demo')
            cmdd = 'git log --author=\'' + author + '\' --pretty=tformat: --numstat | awk \'{ add += $1 ; subs += $2 ' \
                                                    '; loc += $1 - $2 } END ' \
                                                    '{ printf "%s,%s,%s",add,subs,loc }\'>> ~/Desktop/项目-demo_code1.csv'
            p = subprocess.Popen(cmdd, shell=True, cwd='/Users/caining/Documents/demo/spring-boot-demo')
            out, err = p.communicate()
            subprocess.Popen('printf \\"}, >> ~/Desktop/项目-demo_code1.csv', shell=True,
                                      cwd='/Users/caining/Documents/demo/spring-boot-demo')
    subprocess.Popen('printf {}] >> ~/Desktop/项目-demo_code1.csv', shell=True,
                     cwd='/Users/caining/Documents/demo/spring-boot-demo')
                      
```

- 执行完结果如下：

```json
[
    {
        "name":"Yangkai.Shen",
        "score":"124483,76983,47500"
    },
    {
        "name":"yangkai.shen",
        "score":"9099,446,8653"
    },
    {
        "name":"fxbin",
        "score":"2902,216,2686"
    },
  ...
]
```

<img src="https://www.hualigs.cn/image/6091051175592.jpg" alt="image-20210501211114654" style="zoom:50%;" />

- 搭建springBoot 项目<img src="https://www.hualigs.cn/image/609105635b33f.jpg" alt="image-20210502191400391" style="zoom:25%;" />
- 编写upload上传接口，并集成swagger2<img src="https://www.hualigs.cn/image/609105d3e24d9.jpg" alt="image-20210502192113500" style="zoom:100%;" />
- Upload to mysql 保证数据的完整性，对比后正确

<img src="https://www.hualigs.cn/image/6091065edd93a.jpg" alt="image-20210502185428210" style="zoom:50%;" />

- spring to mysql 关键代码如下

```java
try {
            InputStream inputStream = file.getInputStream();
            BufferedReader br = new BufferedReader(new InputStreamReader(inputStream, StandardCharsets.UTF_8));
            String str = null;
            while((str = br.readLine()) != null){
                String[] split = str.split("\n");
                int len = split.length;
                List<GitLog> gitLogList = new ArrayList<>();
                for (int j = 0; j < len; j++) {
                    String[] split1 = split[j].split(",");
                    try {
                        String commit_id_long = split1[1];
                        GitLog oneById = gitDao.findOneById(commit_id_long);
                        GitLog gitLog = new GitLog()
                            .setProject(split1[0].replace("’", ""))
                            .setCommit_id_long(commit_id_long)
                            .setCommit_id(split1[2])
                            .setAuthor(split1[3])
                            .setEmail(split1[4])
                            .setTime_stamp(Long.valueOf(split1[5]))
                            .setTime(split1[6].replace("+0800", ""))
                            .setMessage(split1[7].replace("’", ""));
                        if (oneById != null) {
                            gitDao.update(gitLog);
                        } else {
                            gitLogList.add(gitLog);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
                gitDao.insertList(gitLogList);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
```

- 存入mysql 后，更加方便统计，除了基础代码量统计外，存入mysql 好处如下：
  - 可以统计不通同一个用户不同author情况
  - 可以统计不通email相同author情况
  - 可以统计最晚时间
  - 可以统计当日提交最多提交数等
  - 重复author like this：<img src="https://www.hualigs.cn/image/609107270e15a.jpg" alt="image-20210502190448732" style="zoom:33%;" />
  - 其他统计（全年某日最多提交次数、全年总提交次数、全年提交次数排名、全年某日最晚提交（加班最晚时间点提交）、全年加班提交次数等统计）

```sql
--sql相关脚本略。。。进行中，这里不影响本篇文章。
```

- 总体来说，利用sql 超强统计能力+git-log 本身代码统计能力，可以做到我们想做的项目：【Git代码年报总结】
- 作者个人基本脚本以及简单后端初步形成，后面打算结合前端界面，前端负责**炫酷**，完成本项目，项目规划中，感兴趣同学欢迎一起讨论；come baby！
- 到此项目后端以及准备阶段脚本已初步形成，差具体接口开发以及统计

# 总结

- git log 基础知识，format 很强大，样式也很多样

- git shortlog 基础知识&git blame 基础知识

- 项目实战结合了mysql+spring+boot，方便统计

- git 命令导出脚本运用了python 方便循环导出作者所写代码数量

- mysql 其他脚本进行中

- 后期小程序或者web界面搭建进行中

- 讨论@QiShare


# 引用

> - https://git-scm.com/docs/git-log
> - https://git-scm.com/docs/git-blame
>
> - https://git-scm.com/docs/git-shortlog
>
> - 本项目后端代码&&项目统计源参考：https://github.com/xkcoding/spring-boot-demo