---
layout: post
category : lessons
comments : false
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}


This is an example of a draft. Read more here: [http://jekyllrb.com/docs/drafts/](http://jekyllrb.com/docs/drafts/)

{% highlight ruby linenos %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}

<ul>
    {% for post in site.posts %}
        <li>
            <a href="{{ post.url }}">{{ post.title }}</a>
            <p>{{ post.excerpt }}</p>
        </li>
    {% endfor %}
</ul>
