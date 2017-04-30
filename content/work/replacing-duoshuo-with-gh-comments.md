# 使用 GitHub Comments 替换多说评论

- pubdate: 2017-04-27
- tags: node.js, nico, gh-comments
- gh_issue_id: 1

------

最近多说评论要关闭了，放在 GitHub Pages 上的静态博客需要新的评论功能了。之前有想法使用 [GitHub Issues API](https://developer.github.com/v3/issues/) 来实现一个评论系统，目前 GitHub 已经有[类似实现](https://github.com/imsun/gitment)了，有兴趣的可以尝试一下。

## GitHub Comments

在看到一篇 [Replacing Disqus with GitHub Comments](http://donw.io/post/github-comments/) 文章之后，觉得这个思路更是简单可行。于是动手给博客主题加了一个 GitHub Comments：

1. 首先在博客的 GitHub repo 上为文章创建一个 issue，比如为这篇文章创建的一个 [issue](https://github.com/7anshuai/blog/issues/1)
2. 所有的评论都是在 issue 里面发布
3. 在博客的文章页面添加一些 JavaScript 代码通过 GitHub API 获取到指定 issue 的所有评论并展示

好处是显而易见的：

- 可以使用 [Markdown](http://daringfireball.net/projects/markdown/) 进行评论，支持插入代码，图片，列表等等
- 可以使用 GitHub 消息通知来及时回复评论
- 对于访问博客的读者来说，几乎没有任何追踪代码了
- 可以过滤一部分垃圾评论

最重要的是你不需要申请任何 API 权限去读取 GitHub JSON 数据，它是完全开放的。这篇文章的评论的数据可以在[这里](https://api.github.com/repos/7anshuai/blog/issues/1/comments)获取到。
第一条评论是这样的：

```json
{
    "body": "Inspired by http://donw.io/post/github-comments/.",
    "created_at": "2017-04-27T03:12:41Z",
    "html_url": "https://github.com/7anshuai/blog/issues/1#issuecomment-297599593",
    "id": 297599593,
    "issue_url": "https://api.github.com/repos/7anshuai/blog/issues/1",
    "updated_at": "2017-04-27T03:12:41Z",
    "url": "https://api.github.com/repos/7anshuai/blog/issues/comments/297599593",
    "user": {
        "avatar_url": "https://avatars0.githubusercontent.com/u/9865150?v=3",
        "events_url": "https://api.github.com/users/7anshuai/events{/privacy}",
        "followers_url": "https://api.github.com/users/7anshuai/followers",
        "following_url": "https://api.github.com/users/7anshuai/following{/other_user}",
        "gists_url": "https://api.github.com/users/7anshuai/gists{/gist_id}",
        "gravatar_id": "",
        "html_url": "https://github.com/7anshuai",
        "id": 9865150,
        "login": "7anshuai",
        "organizations_url": "https://api.github.com/users/7anshuai/orgs",
        "received_events_url": "https://api.github.com/users/7anshuai/received_events",
        "repos_url": "https://api.github.com/users/7anshuai/repos",
        "site_admin": false,
        "starred_url": "https://api.github.com/users/7anshuai/starred{/owner}{/repo}",
        "subscriptions_url": "https://api.github.com/users/7anshuai/subscriptions",
        "type": "User",
        "url": "https://api.github.com/users/7anshuai"
    }
}
```

## 代码

我的博客使用 [Nico](https://github.com/lepture/nico) 来生成静态文件。不管你使用什么静态文件生成器，都只需给使用的主题添加一个简单的 `comments.html`，我的看起来是这样:

```html
<div class="gh-comments" data-comment-id="{{post.gh_issue_id}}" data-title="{{ post.title }}"></div>
<script type="text/javascript">
  (function (){
    if (!'{{post.gh_issue_id}}') return false;
    var gh_comments = document.getElementsByClassName('gh-comments')[0];
    var a = document.createElement('a');
    a.href = '{{config.github_issues}}';
    var gh_api = 'https://api.github.com/repos' + a.pathname;
    var gh_issue_id = '{{post.gh_issue_id}}';
    var gh_issue_url = a.href +  '/' + gh_issue_id;
    var gh_comments_url = gh_api + '/' + gh_issue_id + '/comments';
    fetch(gh_comments_url, {
      headers: new Headers({
        'Accept': 'application/vnd.github.v3.html+json',
        'Content-Type': 'application/json'
      }),
      method: 'GET'
    }).then((res) => {
      if (res.status == 200) return res.json();
      let error = new Error('HTTP Exception[GET]');
      error.status = res.status;
      error.statusText = res.statusText;
      error.url = res.url;
      throw error;
    }).then((json) => {
        gh_comments.insertAdjacentHTML('afterbegin', `<h3>GitHub Comments</h3>
          <p>Visit the <a href="${gh_issue_url}">GitHub Issue</a> to comment on this post.</p>`);

        for (let comment of json) {
          let date = new Date(comment.created_at);
          let c = '<div class="gh-comment">' +
              `<img src="${comment.user.avatar_url}" width="24px"> ` +
              `<a href="${comment.user.html_url}">${comment.user.login}</a>` +
              ' posted at ' +
              `<time>${date.toUTCString()}</time>` +
              '<hr>' +
              comment.body_html +
              '</div>';
          gh_comments.insertAdjacentHTML('beforeend', c);
        }
    }).catch((err) => {
      gh_comments.insertAdjacentHTML('afterbegin', `<h3>GitHub Comments</h3>
        <p>Comments are not open for this post yet</p>`);
    });
  })();
</script>
```

其中：

- `{{config.github_issues}}` - 在 nico.json 配置文件中添加的本博客 issues 地址
- `{{post.gh_issue_id}}` - 在文章的 Markdown 文档中指定相应的 issue id
- `{{post.title}}` - 文章的标题（并没有什么用）

> 注意：如果你使用 nico， 默认是不支持读取 `post.gh_issue_id`，找到相应的 [lib/sdk/post.js](https://github.com/lepture/nico/blob/master/lib/sdk/post.js#L98)

> 手动添加一行 `post.gh_issue_id = post.meta.gh_issue_id;`

在主题的 `post.html` 文件中将 `comments.html` 引入进来即可：

```html
{%- if config.github_issues %}
{%- include "_comments.html" %}
{%- endif %}
```

当你在 nico.json 配置好了博客 issues 地址，并在某篇文章指定了 issue id，就能成功组合一个该 issue 的 comments API url，获取到了数据展示在文章后面。

参考链接：
- [Replacing Disqus with GitHub Comments](http://donw.io/post/github-comments/)
