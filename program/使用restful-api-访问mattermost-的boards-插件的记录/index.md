# 使用restful Api 访问mattermost 的boards 插件的记录


安装mattermost 之后， 它会自带一个boards插件， 这个插件允许使用 restful api 做一些自动化操作。 

## 主要内容

### 获取请求插件需要的 csrf token 

调用 Mattermost 的登录api (*/api/v4/users/login*) 时， 附加一个header `X-Requested-With: XMLHttpRequest` ， 在返回值中就会多获得一个 cookie `MMCSRF`，这个cookie 的值就是 请求boards需要的 csrf token

参考解析代码： 

```java 
    private String getCSRFToken(Response response) {
        List&lt;String&gt; cookies = response.headers(&#34;Set-Cookie&#34;);
        String mmcsrf = null;
        for (String cookie : cookies) {
            if (cookie.startsWith(&#34;MMCSRF=&#34;)) {
                // cookie 格式类似于 &#34;MMCSRF=some-real-chars; Path=/; Expires=...&#34;
                // 截取等号后面直到遇到分号之前的部分
                int start = &#34;MMCSRF=&#34;.length();
                int end = cookie.indexOf(&#39;;&#39;, start);
                if (end == -1) {
                    end = cookie.length();
                }
                mmcsrf = cookie.substring(start, end);
                break;
            }
        }

        return mmcsrf;
    }
```

### 使用token 去请求接口

**获取board上的全部cards的接口**

```http
GET ${MATTERMOST_SERVER_URL}/plugins/focalboard/api/v2/boards/${BOARD_ID}/cards?page=0&amp;per_page=100
Authorization: Bearer ${MATTERMOST_TOKEN}
X-CSRF-Token: ${CSRF_TOKEN}
X-Requested-With: XMLHttpRequest
```

要点说明： 
- 使用GET 请求这个地址 
- MATTERMOST_SERVER_URL 是mattermost 的服务器地址
- BOARD_ID 是 看板的id
- MATTERMOST_TOKEN  是 mattermost 用户登录成功的token
- CSRF_TOKEN  是 相同用户上一步 获取到的csrf token

参考结果： 

```json
[
  {
    &#34;id&#34;: &#34;c71g38nnsd7nrpksfyygdnuuj8e&#34;,
    &#34;boardId&#34;: &#34;bceibm6eysi8pjj98tp6581phpy&#34;,
    &#34;createdBy&#34;: &#34;xybegg9i53f4i8gydadyk8neqr&#34;,
    &#34;modifiedBy&#34;: &#34;xybegg9i53f4i8gydadyk8neqr&#34;,
    &#34;title&#34;: &#34;每周一篇博客&#34;,
    &#34;contentOrder&#34;: [],
    &#34;icon&#34;: &#34;&#34;,
    &#34;isTemplate&#34;: false,
    &#34;properties&#34;: {
      &#34;a969ar3tgfkk8ny5o7gdm4n1gbh&#34;: &#34;{\&#34;from\&#34;:1749405600016}&#34;,
      &#34;a9whgdeet9fqtrwrn7ap1r6ph3w&#34;: &#34;akb4d3qaz5srqtuq7u1zpnaegwy&#34;
    },
    &#34;createAt&#34;: 1749405610056,
    &#34;updateAt&#34;: 1749405610056,
    &#34;deleteAt&#34;: 0
  },
  {
    &#34;id&#34;: &#34;c97r7gby6q3g9tfce5qbaqik6kr&#34;,
    &#34;boardId&#34;: &#34;bceibm6eysi8pjj98tp6581phpy&#34;,
    &#34;createdBy&#34;: &#34;xybegg9i53f4i8gydadyk8neqr&#34;,
    &#34;modifiedBy&#34;: &#34;xybegg9i53f4i8gydadyk8neqr&#34;,
    &#34;title&#34;: &#34;每周一篇博客&#34;,
    &#34;contentOrder&#34;: [],
    &#34;icon&#34;: &#34;&#34;,
    &#34;isTemplate&#34;: false,
    &#34;properties&#34;: {
      &#34;a969ar3tgfkk8ny5o7gdm4n1gbh&#34;: &#34;{\&#34;from\&#34;:1750010400014}&#34;,
      &#34;a9whgdeet9fqtrwrn7ap1r6ph3w&#34;: &#34;akb4d3qaz5srqtuq7u1zpnaegwy&#34;
    },
    &#34;createAt&#34;: 1750010410071,
    &#34;updateAt&#34;: 1750010410071,
    &#34;deleteAt&#34;: 0
  },
  {
    &#34;id&#34;: &#34;csxx3ej1qeibiipxxjjddxc6f7w&#34;,
    &#34;boardId&#34;: &#34;bceibm6eysi8pjj98tp6581phpy&#34;,
    &#34;createdBy&#34;: &#34;xybegg9i53f4i8gydadyk8neqr&#34;,
    &#34;modifiedBy&#34;: &#34;xybegg9i53f4i8gydadyk8neqr&#34;,
    &#34;title&#34;: &#34;推进40分钟主要目标&#34;,
    &#34;contentOrder&#34;: [],
    &#34;icon&#34;: &#34;&#34;,
    &#34;isTemplate&#34;: false,
    &#34;properties&#34;: {
      &#34;a969ar3tgfkk8ny5o7gdm4n1gbh&#34;: &#34;{\&#34;from\&#34;:1750600200037}&#34;,
      &#34;a9whgdeet9fqtrwrn7ap1r6ph3w&#34;: &#34;akb4d3qaz5srqtuq7u1zpnaegwy&#34;
    },
    &#34;createAt&#34;: 1750600210040,
    &#34;updateAt&#34;: 1750600210040,
    &#34;deleteAt&#34;: 0
  }
]
```

### 接口文档以及其他地址 

- boards插件是基于 https://github.com/mattermost-community/focalboard  实现的。 
- restful api 的接口文档在 https://htmlpreview.github.io/?https://github.com/mattermost/focalboard/blob/main/server/swagger/docs/html/index.html   
  - 比较简陋， 一些接口的请求体和响应体都没说明， 需要自己测试。。 
- 结构就是 board 包含 cards ， 没有一层 lists 。
- 在插件里面的 Status 会把卡片分成一列又一列， 但是这只是卡片的一个属性而已。  
- 所有的属性， 属性值都有一个id, 调用的时候都是使用id 的。  调用`${MATTERMOST_SERVER_URL}/plugins/focalboard/api/v2/teams/${TEAM_ID}/boards` 会得到一个响应， 响应里面有一个 `cardProperties` 对象， 这个对象记录了所有当前看板的属性。  TEAM_ID 可以从web 界面的 url 上找到。 

### 其他接口示例

**添加card**

```http
POST ${MATTERMOST_SERVER_URL}/plugins/focalboard/api/v2/boards/${BOARD_ID}/cards
Authorization: Bearer ${MATTERMOST_TOKEN}
X-CSRF-Token:  ${CSRF_TOKEN}
X-Requested-With: XMLHttpRequest
Content-Type: application/json
```
*请求体*
```json
{
    &#34;title&#34;: &#34;create-by-api&#34;,
    &#34;icon&#34;: &#34;😒&#34;,
    &#34;isTemplate&#34;: false,
    &#34;properties&#34;: {
      &#34;a9whgdeet9fqtrwrn7ap1r6ph3w&#34;: &#34;akb4d3qaz5srqtuq7u1zpnaegwy&#34;
    }
}
```

**删除card**

```http
DELETE ${MATTERMOST_SERVER_URL}/plugins/focalboard/api/v2/boards/${BOARD_ID}/blocks/${CARD_ID}
Authorization: Bearer ${MATTERMOST_TOKEN}
X-CSRF-Token:  ${CSRF_TOKEN}
X-Requested-With: XMLHttpRequest
Content-Type: application/json
```

这里的路径不使用 cards, 使用 blocks, 以及后面给 CARD_ID



---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/program/%E4%BD%BF%E7%94%A8restful-api-%E8%AE%BF%E9%97%AEmattermost-%E7%9A%84boards-%E6%8F%92%E4%BB%B6%E7%9A%84%E8%AE%B0%E5%BD%95/  

