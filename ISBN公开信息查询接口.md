## ISBN公开信息查询接口

### 背景

我们知道市面上主流的通过isbn 好获取图书信息的接口大部分是收费，图书行业缺乏公共开源的解决方案，各厂商把握资源却缺乏共享精神，拿这个接口收费，图书信息化豆瓣因为种种原因关闭了其图书查询接口，笔者实现了该接口查询能力，现在免费开源，希望大家合理使用，拒绝批量爬取！！

- 通过ISBN 获取图书信息市面上多是收费
- 有能力有资源的大厂也关闭了接口
- 本服务免费提供

### 准备工作

- [免费域名申请](https://nic.ua/en/domains/.pp.ua)
- [免费申请流程参考](https://tlanyan.pp.ua/personal-free-pp-ua-domain-tutorial/#bnp_i_1) 
- [免费证书申请](https://yundun.console.aliyun.com/?spm=5176.14492338.commonbuy2container.dcas_dv_public_cn_ZjqNavbar_product_console.cfbd778bVI6NVL&p=cas#/certExtend/free)
- 服务器地址：152.136.96.242
- 域名：暂无
- 由于是免费服务，服务器配置低，所以请大家合理使用，拒绝批量爬取。

### API使用

#### curl

```bash
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787111605768" -H "accept: */*"
```

#### Request URL

```base
http://152.136.96.242/cnn/v1/book/scan?isbn=9787111453567
```

#### Server response

```json
{
    "code":0,
    "message":"success",
    "data":{
        "id":9787111453567,
        "name":"算法心得：高效算法的奥秘（原书第2版）",
        "subName":"高效算法的奥秘",
        "author":"Henry S. Warren, Jr.",
        "authorIntro":"计算机科学家，在IBM供职50余年，经历了IBM704时代、PowerPC时代及其后种种更迭。曾参与多个军事指挥与控制系统工程，并且参加了由Jack Schwarz领衔的“SET语言”项目。自1973年起，Hank就职于IBM研发部，努力探索编译器和计算机架构。当前正研究一种旨在每秒执行百亿亿次运算的超级计算机。Hank拥有纽约大学柯朗数学科学研究所计算机科学博士学位。",
        "translator":"爱飞翔",
        "translatorIntro":"资深软件开发工程师，擅长Web开发、移动开发和游戏开发，有10余年开发经验，曾主导和参与了多个手机游戏和手机软件项目的开发，经验十分丰富。他是手机软件开发引擎AgileMobileEngine的创始人兼项目经理，同时也是CatEngine手机游戏开发引擎的联合创始人兼代码维护员。他对极限编程、设计模式、重构、测试驱动开发、敏捷软件开发等也有较深入的研究，目前负责敏捷移动开发网（http://www.agilemobidev.com/）的运营。业余爱好文学和历史，有一定的文学造诣。翻译并出版了多本计算机著作。",
        "publishing":"机械工业出版社",
        "published":"2014-3",
        "designed":"平装",
        "isbn":"9787111453567",
        "pages":"419",
        "photoUrl":"https://oslanka.github.io/statichtml.github.io/images/s27240262.jpg",
        "localUrl":"http://152.136.96.242/cnn/v1/book/image/s27240262.jpg",
        "price":"89.00元",
        "description":"在本书中，作者给我们带来了一大批极为诱人的知识，其中包括各种节省程序运行时间的技巧、算法与窍门。学习了这些技术，程序员就可写出优雅高效的软件，同时还能洞悉其中原理。这些技术极为实用，而且其问题本身又非..."
    }
}
```

##### 关于静态资源图片存取

- Github 托管的静态资源长期存储："photoUrl": "https://oslanka.github.io/statichtml.github.io/images/s34003065.jpg",//
-  服务器存储："localUrl": "http://152.136.96.242/cnn/v1/book/image/s34003065.jpg"// 内存有限，量大后可能删除
- 请优先使用gitHub 图片地址

#### 其他ISBN测试

<img src="http://152.136.96.242/cnn/v1/book/image/s29878508.jpg" alt="image-20211215163333735" style="zoom:20%;" />[《复盘+》](http://152.136.96.242/cnn/v1/book/scan?isbn=9787111605768)<img src="http://152.136.96.242/cnn/v1/book/image/s1800355.jpg" alt="image-20211215163333735" style="zoom:30%;" />[《万历十五年》](http://152.136.96.242/cnn/v1/book/scan?isbn=9787108009821)

```bash
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787533945251" -H "accept: */*"
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787115509840" -H "accept: */*"
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787115510167" -H "accept: */*"
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787115476883" -H "accept: */*"
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787115451200" -H "accept: */*"
curl -X GET "http://152.136.96.242/cnn/v1/book/scan?isbn=9787115485885" -H "accept: */*"
```

#### 其他说明

- 笔者本着共享精神，免费提供API 服务，侵删，感谢支持，如有疑问或者接口有bug 请@QiShare
- 后期可能会增加限制规则
