---
theme: gaia
style: |
  section {
    background-color: #ccc;
    font-size: 28px;
    padding: 20px 24px 24px 24px;
  }
  pre {
    margin: 0.5em 0 0 0;
  }
size: 4K
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('./assets/background.jpg')
marp: true
---

![bg opacity:8%](./assets/background-home.jpg)

# **深入浅出 CRUD**
#### 我所理解的业务逻辑

Miclle Zheng
@miclle

-----------------------------------------------------------------------

# Create, Read, Update, and Delete (CRUD)

CRUD     | Databases (SQL)   | MongoDB  | RESTful APIs (HTTP)
:-------:|:-----------------:|:--------:|:--------------------:
Create   | INSERT            | Insert   | POST
Read     | SELECT            | Remove   | GET
Update   | UPDATE            | Find     | PUT / PATCH
Delete   | DELETE            | Update   | DELETE

-----------------------------------------------------------------------

# 来几道简单的面试题：

1. 删除数据用什么方法？
2. PUT *vs* PATCH
3. 200 *vs* 201 *vs* 204
4. 301 *vs* 302
5. 401 *vs* 403

-----------------------------------------------------------------------

# 初探业务系统：从一个 Blog 程序开始
### 文章（Article）模型分析：

<div>
  <div style="width: 50%; float: left">

  字段描述   | 字段名称           | 字段类型
  :-------:|:-----------------:|:--------------
  文章主键   | id               | integer
  文章标题   | title            | string
  文章内容   | content          | string
  创建时间   | created_at       | unix timestamp
  更新时间   | updated_at       | unix timestamp

  </div>
  <div style="width: 50%; float:right; margin-top: 15px">

  ```go
  // Article model
  type Article struct {
    ID        uint   `json:"id"         gorm:"primaryKey"`
    Title     string `json:"title"      gorm:"size:255"`
    Content   string `json:"content"    gorm:"size:1048576"`
    CreatedAt int64  `json:"created_at"`
    UpdatedAt int64  `json:"updated_at"`
  }
  ```

  </div>
</div>

-----------------------------------------------------------------------

### 功能分析

功能描述 | CRUD    | Database  | RESTful APIs (HTTP) | URL
:------:|:-------:|:---------:|:-------------------:|--------------
创建文章 | Create  | INSERT    | POST                | /articles
文章列表 | Read    | SELECT    | GET                 | /articles
文章详情 | Read    | SELECT    | GET                 | /articles/:id
更新文章 | Update  | UPDATE    | PUT / PATCH         | /articles/:id
删除文章 | Delete  | DELETE    | DELETE              | /articles/:id

-----------------------------------------------------------------------

### RESTful API

#### Create an article
```
POST /articles
```

###### Parameters
Name    | Type   | In    | Description
:------:|:------:|:-----:|------------
title   | string | body  | **Required**. The title of the article.
content | string | body  | **Required**. The contents of the article.

###### Code samples

```shell
curl \
  -X POST \
  -H 'Accept: application/json, text/plain, */*' \
  http://localhost:3000/articles \
  -d '{"title":"title", "content":"content"}'
```

```
Status: 201 Created
```

-----------------------------------------------------------------------

#### List articles
```
GET /articles
```

##### Parameters
Name     | Type    | In     | Description
:-------:|:-------:|:------:|------------
per_page | integer | query  | Results per page.
page     | integer | query  | Page number of the results to fetch.
q        | string  | query  |

###### Code samples

```shell
curl 'http://localhost:3000/articles' \
  -H 'Accept: application/json, text/plain, */*' \
```

-----------------------------------------------------------------------

###### Response
```
Status: 200 OK
```

<div>
  <div style="width: 48%; float: left; margin-right: 5px;">

  ```
  [
    {
      "id": 1,
      "title": "title",
      "content": "content",
      "created_at": 1627290338,
      "updated_at": 1627290338
    }
    ...
    ...
    ...
  ]
  ```

  </div>

  <div style="width:3%; float: left; margin-top: 100px; text-align: center;">

  *vs*

  </div>

  <div style="width: 48%; float:right; margin-left: 5px;">

  ```
  {
    items: [
      {
        "id": 1,
        "title": "title",
        "content": "content",
        "created_at": 1627290338,
        "updated_at": 1627290338
      }
      ...
    ]
  }
  ```

  </div>
</div>

-----------------------------------------------------------------------

#### Get an articles
```
GET /articles/:id
```

###### Code samples

```shell
curl 'http://localhost:3000/articles/1' \
  -H 'Accept: application/json, text/plain, */*' \
```

###### Response
```
Status: 200 OK
```

```
{
  "id": 1,
  "title": "title",
  "content": "content",
  "created_at": 1627290338,
  "updated_at": 1627290338
}
```

-----------------------------------------------------------------------
#### Update an article
```
PATCH /articles/:id
```

###### Parameters
Name    | Type    | In   | Description
:------:|:-------:|:----:|-----------------------------
id      | integer | path | The article id parameter
title   | string  | body | The title of the article.
content | string  | body | The contents of the article.

###### Code samples

```shell
curl 'http://localhost:3000/articles/1' \
  -X 'PATCH' \
  -H 'Accept: application/json, text/plain, */*' \
  -H 'Content-Type: application/json;charset=UTF-8' \
  --data-raw '{"title":"title", "content":"content"}' \
```

```
Status: 200 OK
```

-----------------------------------------------------------------------

#### Delete an article
```
DELETE /articles/:id
```

###### Parameters
Name    | Type    | In   | Description
:------:|:-------:|:----:|-----------------------------
id      | integer | path | The article id parameter
title   | string  | body | The title of the article.
content | string  | body | The contents of the article.

###### Code samples

```shell
curl 'http://localhost:3000/articles/1' \
  -X 'DELETE' \
```

```
Status: 204 No Content
```

-----------------------------------------------------------------------

# 多租户（有多个作者） Blog 怎么处理？

#### 加两个字段还是加个模型？

<div>
  <div style="width: 48%; float: left; margin-right: 5px;">

  ```go
  // Article model
  type Article struct {
    ID          uint   `json:"id"           gorm:"primaryKey"`
    Title       string `json:"title"        gorm:"size:255"`
    AuthorName  string `json:"author_name"  gorm:"size:255"`
    AuthorTitle string `json:"author_title" gorm:"size:255"`
    Summary     string `json:"summary"      gorm:"size:65535"`
    Content     string `json:"content"      gorm:"size:1048576"`
    CreatedAt   int64  `json:"created_at"`
    UpdatedAt   int64  `json:"updated_at"`
  }
  ```

  </div>

  <div style="width:3%; float: left; margin-top: 100px; text-align: center;">

  *vs*

  </div>

  <div style="width: 48%; float:right; margin-left: 5px;">

  ```go
  // User model
  type User struct {
    ID        uint   `json:"id"         gorm:"primaryKey"`
    Name      string `json:"name"       gorm:"size:255"`
    Title     string `json:"title"      gorm:"size:255"`
    Bio       string `json:"bio"        gorm:"size:65535"`
    CreatedAt int64  `json:"created_at"`
    UpdatedAt int64  `json:"updated_at"`
  }

  // Article model
  type Article struct {
    ID        uint   `json:"id"         gorm:"primaryKey"`
    UserID    uint   `json:"-"          gorm:"index"`
    Title     string `json:"title"      gorm:"size:255"`
    Summary   string `json:"summary"    gorm:"size:65535"`
    Content   string `json:"content"    gorm:"size:1048576"`
    CreatedAt int64  `json:"created_at"`
    UpdatedAt int64  `json:"updated_at"`

    Author *User `json:"author" gorm:"foreignKey:UserID"`
  }
  ```

  </div>
</div>

-----------------------------------------------------------------------

#### List articles
```
GET /articles
```

###### Response

```
{
  items: [
    {
      "id": 1,
      "title": "title",
      "content": "content",
      "created_at": 1627290338,
      "updated_at": 1627290338,
      "author": {
        "id": "1",
        "name": "Tom",
        "title": "CTO",
        "created_at": 1627290338,
        "updated_at": 1627290338
      }
    }
    ...
  ]
}
```

-----------------------------------------------------------------------

# 给文章加点评论

### 评论（Comment）模型分析：

<div>
  <div style="width: 50%; float: left">

  字段描述   | 字段名称      | 字段类型
  :-------:|:------------:|:--------------
  评论主键   | id          | integer
  文章外键   | article_id  | integer
  评论内容   | content     | string
  创建时间   | created_at  | unix timestamp
  更新时间   | updated_at  | unix timestamp

  </div>
  <div style="width: 50%; float:right; margin-top: 15px">

  ```go
  // Comment model
  type Comment struct {
    ID        uint   `json:"id"         gorm:"primaryKey"`
    ArticleID uint   `json:"article_id" gorm:"index"`
    Content   string `json:"content"    gorm:"size:1048576"`
    CreatedAt int64  `json:"created_at"`
    UpdatedAt int64  `json:"updated_at"`
  }
  ```

  </div>
</div>

-----------------------------------------------------------------------
