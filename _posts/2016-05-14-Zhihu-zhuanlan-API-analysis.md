---
layout: post # needs to be post
title: 知乎专栏 API 分析
summary: 知乎专栏 API 分析
featured-img: raindrop_glass
categories: [Note]
---

## 声明
声明：以下所有 API 均由 **知乎(Zhihu.INC)** 提供，本人通过非正常手段获取。获取与共享的行为有侵犯知乎权益的嫌疑。若被告知停止使用与共享，本人将及时删除整个项目。请您知悉相关情况，遵守知乎的协议。API 仅供学习交流使用，请勿用作商业用途。

## 说明
+ 专栏的消息均以 JSON 格式输出;
+ http method 均为 GET;
+ JSON ARRAY 中的数据;只选取了具有代表性的部分进行展示.

## 分析
### 获取指定专栏的信息
```
https://zhuanlan.zhihu.com/api/columns/专栏名
```

例如获取知乎小管家的专栏信息

```
https://zhuanlan.zhihu.com/api/columns/zhihuadmin
```

得到的信息为：

```json
{
  "followersCount": 7883,
  "creator":
    {
      "profileUrl": "https://www.zhihu.com/people/zhihuadmin",
      "bio": "欢迎反馈问题和建议！",
      "hash": "3d198a56310c02c4a83efb9f4a4c027e",
      "uid": 53253479858176,
      "isOrg": false,
      "description": "",
      "slug": "zhihuadmin",
      "avatar":
        {
          "id": "34bf96bf5584ac4b5264bd7ed4fdbc5a",
          "template": "https://pic3.zhimg.com/{id}_{size}.jpg"
        },
      "name": "知乎小管家"
    },
    "topics":
      [
        {
          "url": "https://www.zhihu.com/topic/19552112",
          "id": "19552112",
          "name": "知乎建议反馈"
        },
      ],
    "activateState": "activated",
    "href": "/api/columns/zhihuadmin",
    "acceptSubmission": true,
    "firstTime": false,
    "postTopics":
      [
        {
          "postsCount": 4,
          "id": 2,
          "name": "知乎"
        },
      ],
    "pendingName": "",
    "avatar":
      {
        "id": "5a1ec50171767c4fc856f430e46297db",
        "template": "https://pic4.zhimg.com/{id}_{size}.jpg"
      },
    "canManage": false,
    "description": "知乎社区管理团队官方专栏，不定期更新社区管理工作相关的最新消息。",  
    "pendingTopics": [],
    "nameCanEditUntil": 0,
    "reason": "",
    "banUntil": 0,
    "slug": "zhihuadmin",
    "name": "知乎小管家说",
    "url": "/zhihuadmin",
    "intro": "知乎社区管理团队官方专栏，不定期更新社区管理工作…",  
    "topicsCanEditUntil": 0,
    "activateAuthorRequested": "none",
    "commentPermission": "anyone",
    "following": false,
    "postsCount": 19,  
    "canPost": false
  }
```

+ followersCount: 关注该专栏的人数
+ creator: 该专栏的创建者信息
	- profileUrl: 知乎网的个人主页url
	- bio: 官网信息中的一句话描述
	- hash: hash值
	- uid: uid
	- isOrg: 是否为机构帐号
	- description: 描述
	- slug: slug
	- avatar: 头像信息
		+ id: id值，在拼接url时用到
		+ template: url拼接模版
	- name: 作者名称
+ topics: 专栏话题信息
	- url: 话题在知乎官网中的地址
	- id: 话题id
	- name: 话题名称
+ activateState: 状态信息，是否激活
+ href: api请求地址
+ acceptSubmission: 是否接受提交文章
+ firstTime: 是否为首次提交？？？
+ postTopics: 文章话题信息
	- postsCount: 该话题下文章数
	- id: id
	- name: 话题名称
+ pendingName: ？？？
+ avatar: 专栏头像信息
	- id: 头像id
	- template: url拼接模版
+ canManage: 是否可以进行管理
+ description: 专栏信息描述
+ pendingTopics: ???话题
+ nameCanEditUntil: 距下一次修改名称的时间
+ reason: 被封禁的原因
+ banUntil: 被封禁的时间
+ slug: 专栏slug
+ url: url
+ intro: 专栏介绍信息
+ topicsCanEditUntil: 距下一次修改话题的时间
+ activateAuthorRequested: ???
+ commentPermission: 评论权限
+ following: 是否关注改专栏
+ postsCount: 专栏文章数
+ canPost: 当前帐号是否有发表文章的权限

在获取图片url时，可以根据需要选择不同的尺寸。如下所示，图片质量依次增加。也可以不添加尺寸参数，默认获取最大类型。

```
https://pic3.zhimg.com/34bf96bf5584ac4b5264bd7ed4fdbc5a_s.jpg
https://pic3.zhimg.com/34bf96bf5584ac4b5264bd7ed4fdbc5a_m.jpg
https://pic3.zhimg.com/34bf96bf5584ac4b5264bd7ed4fdbc5a_l.jpg
https://pic3.zhimg.com/34bf96bf5584ac4b5264bd7ed4fdbc5a_b.jpg
https://pic3.zhimg.com/34bf96bf5584ac4b5264bd7ed4fdbc5a_r.jpg
```

### 获取指定专栏的文章列表
```
https://zhuanlan.zhihu.com/api/columns/专栏名/posts?limit=数量&offset=从哪里开始
```

例如，获取知乎小管家的最新的10篇文章

```
https://zhuanlan.zhihu.com/api/columns/zhihuadmin/posts?limit=10&offset=0
```

其中，limit为获取的数量限制，offset为偏移量，即从哪里开始获取。默认数据为limit=10，offset=0，当然你也可以根据需要定义二者的值。

获得的数据为:

```json
[
  {  
    "isTitleImageFullScreen": false,
    "rating": "none",
    "sourceUrl": "",
    "publishedTime": "2016-10-13T14:00:49+08:00",
    "links":
      {
        "comments": "/api/posts/22921645/comments"
      },
    "author":
      {
        "profileUrl": "https://www.zhihu.com/people/zhihuadmin",
        "bio": "欢迎反馈问题和建议！",
        "hash": "3d198a56310c02c4a83efb9f4a4c027e",
        "uid": 53253479858176,
        "isOrg": false,
        "description": "",  
        "slug": "zhihuadmin",
        "avatar":
          {
            "id": "34bf96bf5584ac4b5264bd7ed4fdbc5a",
            "template": "https://pic3.zhimg.com/{id}_{size}.jpg"
          },
        "name": "知乎小管家"
      },
    "url": "/p/22921645",
    "title": "新版社区管理规定于 2016 年 10 月 13 日正式运行",
    "titleImage": "https://pic2.zhimg.com/v2-517f47cc5269b57f26ea4a25f29d8505_r.jpg",
    "summary": "",
    "content": "<p>2016 年 9 月 13 日小管家发布了新版社区管理规定，明确了「恶意行为」的定义和违规类型，同时增加了「发布垃圾广告信息」的 3 种违规类型。在完成了为期 1 个月的试运行后<b>，</b><b><a href=\"https://www.zhihu.com/question/19790711/answer/36685915\" class=\"\" data-editable=\"true\" data-title=\"\u65b0\u7248\u793e\u533a\u7ba1\u7406\u89c4\u5b9a\">\u65b0\u7248\u793e\u533a\u7ba1\u7406\u89c4\u5b9a</a>",
    "state": "published",
    "href": "/api/posts/22921645",
    "meta":
      {
        "previous": null,
        "next": null
      },
    "commentPermission": "anyone",
    "snapshotUrl": "",
    "canComment": false,
    "slug": 22921645,
    "commentsCount": 76,
    "likesCount": 436
  },
]
```

+ isTitleImageFullScreen: 文章标题大图是否全屏
+ rating: ???评级
+ sourceUrl: 源路径
+ publishedTime: 发表时间
+ links: 链接信息
	- comments: 评论地址
+ creator: 该专栏的创建者信息
	- profileUrl: 知乎网的个人主页url
	- bio: 官网信息中的一句话描述
	- hash: hash值
	- uid: uid
	- isOrg: 是否为机构帐号
	- description: 描述
	- slug: slug
	- avatar: 头像信息
		+ id: id值，在拼接url时用到
		+ template: url拼接模版
	- name: 作者名称
+ url: 文章网页内容获取(https://zhuanlan.zhihu.com + url)
+ title: 文章标题
+ titleImage: 文章标题大图url(需要注意的是,titleImage的值有可能为空)，和头像一样，也可以组合不同的尺寸参数获取不同尺寸的图片
+ summary: 文章简要信息
+ content: HTML格式的文章内容详情，可以通过WebView或者UIWebView展示内容
+ state: 文章状态(是否发表)
+ href: api请求地址

### 获取特定的文章信息
```
https://zhuanlan.zhihu.com/api/posts/SLUG
```

例如，获取知乎小管家的最新一篇文章，通过已经获得的信息，有两种方法组合url进行获取，一是通过href值，另一种就是slug值，当然，这两种方式殊途同归。

```
https://zhuanlan.zhihu.com/api/posts/22921645
```

获取到的信息为:

```json
{
  "isTitleImageFullScreen": false,
  "rating": "none",
  "titleImage": "https://pic2.zhimg.com/v2-cab0719fdec0816c475d7b70e016cbb1_r.jpg",
  "links":
    {
      "comments": "/api/posts/22591792/comments"
    },
  "reviewers": [],
  "topics":
    [
      {
        "url": "https://www.zhihu.com/topic/19550887",
        "id": "19550887",
        "name": "知乎社区"
      },
    ],
  "titleImageSize":
    {
      "width": 0,
      "height": 0
    },
  "href": "/api/posts/22591792",
  "excerptTitle": "",
  "author":
    {
      "profileUrl": "https://www.zhihu.com/people/zhihuadmin",
      "bio": "欢迎反馈问题和建议！",
      "hash": "3d198a56310c02c4a83efb9f4a4c027e",
      "uid": 53253479858176,
      "isOrg": false,
      "description": "",
      "badge":
        {
          "identity":
            {
              "description": "知乎官方帐号"
            },
          "best_answerer": null
        },
      "slug": "zhihuadmin",
      "avatar":
        {
          "id": "34bf96bf5584ac4b5264bd7ed4fdbc5a",
          "template": "https://pic3.zhimg.com/{id}_{size}.jpg"
        },
      "name": "知乎小管家"
    },
  "column":
    {
      "slug": "zhihuadmin",
      "name": "知乎小管家说"
    },
  "tipjarState": "inactivated",
  "content": "<p>今天，小管家和大家分享一下站内「不规范转载」行为的处理情况。</p><br>",
  "state": "published",
  "sourceUrl": "",
  "pageCommentsCount": 34,
  "canComment": true,
  "snapshotUrl": "",
  "slug": 22591792,
  "publishedTime": "2016-09-23T17:33:56+08:00",
  "url": "/p/22591792",
  "title": "知乎小管家工作笔记：不规范转载？不可以！",
  "lastestLikers":
    [
      {
        "profileUrl": "https://www.zhihu.com/people/su-shu-95-98",
        "bio": "一个无法用一句话描述的人",
        "hash": "a6975d974fb22609a31dbbdec6f375e3",
        "uid": 657690756534505472,
        "isOrg": false,
        "description": "你说什么都对。",
        "slug": "su-shu-95-98",
        "avatar":
          {
            "id": "888129f4afd91083c0cfd156e6889438",
            "template": "https://pic1.zhimg.com/{id}_{size}.jpg"
          },
        "name": "苏舒"
      },
    ],
  "summary": "<img src=\"https://pic1.zhimg.com/v2-4f2228e8f8292ec30c373488afa2bd48_200x112.jpg\" data-rawwidth=\"920\" data-rawheight=\"638\" class=\"origin_image inline-img zh-lightbox-thumb\" data-original=\"https://pic1.zhimg.com/v2-4f2228e8f8292ec30c373488afa2bd48_r.jpg\">今天，小管家和大家分享一下站内「不规范转载」行为的处理情况。 年初我们发布了...",
  "reviewingCommentsCount": 0,
  "meta":
    {
      "previous":
        {
          "isTitleImageFullScreen": false,
          "rating": "none",
          "titleImage": "https://pic2.zhimg.com/517f47cc5269b57f26ea4a25f29d8505_r.jpg",
          "links":
            {
              "comments": "/api/posts/22434253/comments"
            },
          "topics":
            [
              {
                "url": "https://www.zhihu.com/topic/19550887",
                "id": "19550887",
                "name": "知乎社区"
              },            
            ],
          "href": "/api/posts/22434253",
          "excerptTitle": "",
          "author":
            {
              "profileUrl": "https://www.zhihu.com/people/zhihuadmin",
              "bio": "欢迎反馈问题和建议！",
              "hash": "3d198a56310c02c4a83efb9f4a4c027e",
              "uid": 53253479858176,
              "isOrg": false,
              "description": "",
              "slug": "zhihuadmin",
              "avatar":
                {
                  "id": "34bf96bf5584ac4b5264bd7ed4fdbc5a",
                  "template": "https://pic3.zhimg.com/{id}_{size}.jpg"
                },
              "name": "知乎小管家"
            },
          "column":
            {
              "slug": "zhihuadmin",
              "name": "知乎小管家说"
            },
          "content": "题下发布相同回答；</p><img src=\"https://pic2.zhimg.com/a86dbedbeb67a26b33252ee4e8e886a9_b.png\" data-rawwidth=\"469\" data-rawheight=\"348\" class=\"origin_image zh-lightbox-thumb\" width=\"469\" data-original=\"https://pic2.zhimg.com/a86dbedbeb67a26b33252ee4e8e886a9_r.png\"><br>",
          "state": "published",
          "sourceUrl": "",
          "pageCommentsCount": 0,
          "canComment": true,
          "snapshotUrl": "",
          "slug": 22434253,
          "publishedTime": "2016-09-13T17:34:29+08:00",
          "url": "/p/22434253",
          "title": "新版社区管理规范于今日试运行",
          "summary": "社区管理规范是知友共识的总结，也是社区健康发展的基石。近期，我们不断收到知友们反馈新的扰乱社区秩序的行为，如集赞爆照、点赞抽奖、评论区发布乱码、刷赞刷粉、恶意营销、回答隐藏淘宝链接...",
          "reviewingCommentsCount": 0,
          "meta":
            {
              "previous": null,
              "next": null
            },
          "commentPermission": "anyone",
          "commentsCount": 0,
          "likesCount": 0
        },
      "next": null
    },
  "commentPermission": "anyone",
  "commentsCount": 34,
  "likesCount": 241
}
```

+ isTitleImageFullScreen: 文章标题大图是否全屏
+ rating: ???评级
+ titleImage: 文章标题大图url(需要注意的是,titleImage的值有可能为空)，和头像一样，也可以组合不同的尺寸参数获取不同尺寸的图片
+ links: 链接信息
	- comments: 评论获取地址
+ reviewers: 复审人(在我请求的这么多次中，并没有成功获得过此项对应的值。不排除此项返回具体值的可能性。推测为creator类型。)
+ topics: 专栏话题信息
	- url: 话题在知乎官网中的地址
	- id: 话题id
	- name: 话题名称
+ titleImageSize:
	- width 宽度
	- height 高度
+ href: api请求地址
+ excerptTitle: 引用(摘录)标题
+ author:
	- profileUrl: 知乎网的个人主页url
	- bio: 官网信息中的一句话描述
	- hash: hash值
	- uid: uid
	- isOrg: 是否为机构帐号
	- description: 描述
	- badge 徽章，即认证信息
		+ identity 官方帐号
			- description 认证详情描述
		+ best_answerer 优秀回答者
	- slug: slug
	- avatar: 头像信息
		+ id: id值，在拼接url时用到
		+ template: url拼接模版
	- name: 作者名称
+ column: 所属专栏
	- slug: slug
	- name: 专栏名称
+ tipjarState: 打赏信息
+ content: HTML格式的内容信息详情
+ state: 文章状态，是否发布
+ sourceUrl: 源地址
+ pageCommentsCount: 评论数
+ canComment: 是否可以评论
+ snapshotUrl: 短网址?
+ slug: 文章的slug
+ publishedTime: 文章发表时间
+ url: url地址
+ title: 文章标题
+ lastestLikers: 最近点赞的用户
	- profileUrl: 知乎网的个人主页url
	- bio: 官网信息中的一句话描述
	- hash: hash值
	- uid: uid
	- isOrg: 是否为机构帐号
	- description: 描述
	- slug: slug
	- avatar: 头像信息
		+ id: id值，在拼接url时用到
		+ template: url拼接模版
	- name: 作者名称
+ summary: 文章简要信息
+ reviewingCommentsCount: 审查中的评论数量
+ meta: meta
	- previous 上一篇文章的信息，json结构和上面的内容相同
	- next 下一篇文章的信息
+ commentPermission: 评论权限
+ commentsCount: 评论数量
+ likesCount: 赞的数量

在认证信息中，目前只发现有两种类型，identity官方帐号和best_answerer优秀回答者，由于知乎小管家只是官方帐号，并不是优秀回答者，这里选取优秀回答者[苏莉安](https://www.zhihu.com/people/aton)的信息进行分析。

```json
"badge":
  {
    "identity": null,
    "best_answerer":
      {
        "topics": [],
        "description": "优秀回答者"
      }
  }
```

+ identity 官方帐号
+ best_answerer 优秀回答者
	- topics 话题
	- description 描述信息

### 获取评论列表
```
https://zhuanlan.zhihu.com/api/posts/文章slug/comments?limit=数量限制&offset=偏移量
```

这里拼接url的方法和获取文章列表时的方法时一样的，不再赘述。例如获取文章slug为22591792的前30条评论。

```
https://zhuanlan.zhihu.com/api/posts/22591792/comments?limit=30&offset=0
```

获取到的信息为:

```json
[
  {
    "content": "小管家棒棒哒！支持~",
    "liked": false,
    "href": "/api/posts/22591792/comments/169039008",
    "inReplyToCommentId": 0,
    "reviewing": false,
    "author":
      {
        "profileUrl": "https://www.zhihu.com/people/fei-xiao-gui", "bio": "心中有爱 脚下有风 所以我跑得快~",
        "hash": "41cd5c7249081389da3a523b3cb0629b", "uid": 28048833380352,
        "isOrg": false,
        "description": "品学兼优~",
        "slug": "fei-xiao-gui",
        "avatar":
          {
            "id": "a061ebdd61bb57c81b3206dda51418fc", "template": "https://pic1.zhimg.com/{id}_{size}.jpg"
          },
        "name": "菲小桂"
      },
    "createdTime": "2016-09-23T17:35:54+08:00",
    "featured": false,
    "id": 169039008,
    "likesCount": 1
  }
]
```

+ content: 评论内容
+ liked: 是否给这条评论点过赞
+ href: 该条评论的详情地址
+ inReplyToCommentId: 所回复的评论的id
+ reviewing: 是否正在被审查中
+ author: 评论的作者
	- profileUrl: 知乎网的个人主页url
	- bio: 官网信息中的一句话描述
	- hash: hash值
	- uid: uid
	- isOrg: 是否为机构帐号
	- description: 描述
	- slug: slug
	- avatar: 头像信息
		+ id: id值，在拼接url时用到
		+ template: url拼接模版
	- name: 作者名称
+ createdTime: 评论创建时间
+ featured: ???是否为精彩评论
+ id: 该条评论的id
+ likesCount: 获得的赞的数量

对他人评论的回复数据有些许的不同，inReplyToCommentId的值不为0，json数据中增加了inReplyToUser字段，内容为author类型。这里不再具体分析。

### 获取点赞信息
```
https://zhuanlan.zhihu.com/api/posts/22591792/likers?limit=获取数量&offset=偏移量
```

例如，获取文章slug为22591792的最新20个点赞的人的信息，author类型。

```
https://zhuanlan.zhihu.com/api/posts/22591792/likers?limit=20&offset=0
```

url的拼接方法和上面获取文章信息时方法相同，不再赘述。