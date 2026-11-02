# Writing this blog

This blog is made with [BareBlog](https://github.com/avanelsas/bareblog).

## Where things go

- `posts/`: the published posts, a folder each with `post.md` and its pictures, and
  `comments.edn` with its comments once it has some
- `drafts/`: posts that are not published yet. Git keeps them out.
- `pages/`: pages that are no posts, such as About, a folder each with `page.md`
- `blog.edn`: the settings of the blog
- `series.edn`: the series of posts, written by `bareblog series`
- `fonts/`: the font of the blog, written by `bareblog font`
- `docs/`: the published blog, written by BareBlog. Do not edit it.

## Commands

| You want to | Do this |
|---|---|
| Write a new post | `bareblog new "A title"`, then write in `drafts/a-title/post.md` |
| Add a page, such as About | `bareblog new --page "About"` |
| See the blog, drafts included | `bareblog preview` |
| Publish a post, or a change to one | `bareblog publish <folder>` |
| Publish everything again | `bareblog publish --all` |
| See every tag, or write one tag one way | `bareblog tags`, `bareblog tags rename old "new"` |
| See every series, or make or change one | `bareblog series`, `bareblog series --name "A series" "A post" "Another post"` |
| Remove a series, and keep its posts | `bareblog series --name "A series"` |
| Use a font of Google Fonts, or none | `bareblog font "Name"`, `bareblog font --none` |
| Let readers comment under posts | `bareblog comments connect`, as [COMMENTS.md](https://github.com/avanelsas/bareblog/blob/main/COMMENTS.md) says |
| Read and approve the comments that wait | `bareblog comments` |
| Use the latest release of BareBlog | `bareblog update`, then `bareblog publish --all` |

`bareblog help` lists every command. How a post is written is in BareBlog's
[guide](https://github.com/avanelsas/bareblog/blob/main/GUIDE.md).
