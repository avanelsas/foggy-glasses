 # I created this because I wanted to

![Foggy Glasses](foggy_glasses.png)

Looking at the front page of Hacker news nowadays it seems almost anything is possible now with LLM supported development. As development is costing less effort (whether it is better depends on who you talk to), it now seems anything is getting build, often with a motivation of *"because I can"*.

The problem with that is of course that just because something can be built does not necessarily mean it provides anyone any value. I do not want to imagine the number of tokens burned on things built that are never touched. It is probably a daunting amount, and someone is paying for that.

## Writing is not a solved problem

I know that LLMs can spit out any text you can or cannot imagine. That is not the point of writing though, at least not for me. I'm not a great writer, but I've been blogging for a long time, mostly to make sense of the world around me. By writing things down I question, I learn, I try to make sense of things. An LLM cannot do that for me.

I started blogging at a time it seemed almost anyone was blogging (2007 if I remember correctly). Even a nobody like me could write something useful that could get picked up and end up on TechMeme, instead of all the professional publications we see there now. It was a great time, and I read an enormous amount of other people's blogs too. I wrote for almost 10 years but at some point I ran out of words and stopped.

Recently, I started thinking about picking it up again. My old blog was at Wordpress, and I didn't feel like diving into that platform again. I want something simple, easily configurable, and free in use. I looked around a bit, found several simple solutions that could work for me, but then it occurred to me I still needed a project to use some other things I had created before (notably a web component library) to see how that would work in a live situation.

I thought about it for a while and decided I wanted to build my own blog solution, not because it was possible, but because I wanted (to use) it. And if I did that, I might as well create it in such a way that using it feels simple and fun.

## Requirements

Nothing fancy, I figured I might as well use Github pages and push something there. A few default templates, a bit of theming, images, and the possibility to add animations. Preferably run from the command line and using markup for the blog posts themselves. I looked around, saw a few good solutions, but decided that using my own projects to create something new was more important

## Bareblog

So here it is. It isn't the prettiest, best or most flexible blog solution around. But it does everything I want it to do, and it can be extended if I think of new features. It only has a few commands:

```
bareblog init "My blog"           a new blog
bareblog new "A new blog post"    a draft folder with post.md
             --page "About"       a page that is no post, with page.md
bareblog preview [port]           the whole blog, in the browser
                 --no-open        without opening the browser
bareblog publish a-new-blog-post  one post or page: new, edited or removed
bareblog publish --all            every post and page, as it is on disk
                 --yes            with a yes to every question
bareblog tags                     every tag, and each written in more than one way
bareblog tags rename old "new"    a tag, in every post and page
              --yes               with a yes to the question
bareblog version                  the version of BareBlog
bareblog update
```

I can now create a blog, push it, create a draft post and when done publish it. There is a single `blog.edn` file that offers some configuration, like a template or theme selection, and that's about it. It comes with animations (if you want them), and nice gaussian blurring backgrounds to bring some color to the blog.

## Foggy glasses

The blog I now use is called **Foggy Glasses**. It is a metaphor for a person (like me) that needs glasses to see. Once they are fogged, they rob you of a clear view. And in that moment, when things are unclear, new thoughts appear. That is the best moment perhaps to think and write. I'm not sure where it will lead me. I've written a few things about some tech I use, but also about more personal interests.

I built this for myself, because this is what I wanted. It uses mostly web components from my [web components library](https://github.com/avanelsas/baredom). And while building this I fixed some issues in that library, so that was a win.

Feel free to give BareBlog a try, I am quite happy with it. It is simple enough and I intend to keep blogging for a while, perhaps until I run out of words again.

#BareBlog #writing #blogtool #BareDOM
