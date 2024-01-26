---
layout: post
title: How Gustave works
subtitle: Strategy and details 
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
comments: true
author: Van David LE
---

{: .box-success}
Gustave is an ev3 mindstorm robot designed during the OS course. It's purpose is to be put in an arena against another robot. Each robot needs to catch the flag situated in the other part of the arena. 

## Architecture of Gustave

Front:

![Crepe](https://beautifuljekyll.com/assets/img/crepe.jpg)

Side:

![Crepe](https://beautifuljekyll.com/assets/img/crepe.jpg)

Back:

![Crepe](https://beautifuljekyll.com/assets/img/crepe.jpg)

Top:

![Crepe](https://beautifuljekyll.com/assets/img/crepe.jpg)


Here's a code chunk:

~~~
var foo = function(x) {
  return(x + 5);
}
foo(3)
~~~

And here is the same code with syntax highlighting:

```javascript
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

And here is the same code yet again but with line numbers:

{% highlight javascript linenos %}
var foo = function(x) {
  return(x + 5);
}
foo(3)
{% endhighlight %}

## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.

## Local URLs in project sites {#local-urls}

When hosting a *project site* on GitHub Pages (for example, `https://USERNAME.github.io/MyProject`), URLs that begin with `/` and refer to local files may not work correctly due to how the root URL (`/`) is interpreted by GitHub Pages. You can read more about it [in the FAQ](https://beautifuljekyll.com/faq/#links-in-project-page). To demonstrate the issue, the following local image will be broken **if your site is a project site:**

![Crepe](/assets/img/crepe.jpg)

If the above image is broken, then you'll need to follow the instructions [in the FAQ](https://beautifuljekyll.com/faq/#links-in-project-page). Here is proof that it can be fixed:

![Crepe]({{ '/assets/img/crepe.jpg' | relative_url }})
