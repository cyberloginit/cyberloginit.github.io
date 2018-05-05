---
layout: post
title: GitHub Pages Jekyll Minima Customize Footer
---

# GitHub Pages Jekyll Minima Customize Footer

>[Minima](https://github.com/jekyll/minima) is a one-size-fits-all Jekyll theme for writers. It's Jekyll's default (and first) theme.

As the name implies, Minima offers minimum features by default.

To customize the footer, you have to override its default structure.

At the root of your site, first, create an `_includes` directory and copy `footer.html` from the [Minima GitHub repository](https://github.com/jekyll/minima) into it.

Then customize `footer.html` to your own needs.

```
<footer class="site-footer h-card">
  <data class="u-url" href="{{ "/" | relative_url }}"></data>

  <div class="wrapper">

    <h2 class="footer-heading">{{ site.title | escape }}</h2>

    <div class="footer-col-wrapper">
      <div class="footer-col footer-col-1">
        <ul class="contact-list">
          <li class="p-name">
            {%- if site.author -%}
              {{ site.author | escape }}
            {%- else -%}
              {{ site.title | escape }}
            {%- endif -%}
            </li>
            {%- if site.email -%}
            <li><a class="u-email" href="mailto:{{ site.email }}">{{ site.email }}</a></li>
            {%- endif -%}
        </ul>
      <a href="https://m.do.co/c/de1b31930850" target="_blank"><img src="/images/DO_Powered_by_Badge_black.svg" ></a>
      </div>

      <div class="footer-col footer-col-2">
        <ul style="list-style-type:none">
          <li>Friends: </li>
          <li><a href="https://blog.alalin.me/" target="_blank">alalin.me</a></li>
        </ul>
      </div>

      <div class="footer-col footer-col-3">
        <p>{{- site.description | escape -}}</p>
      </div>
    </div>

  </div>

</footer>
```
In my case, the social networks column is not needed, so I used it for link exchange.

```
<div class="footer-col footer-col-2">
  <ul style="list-style-type:none">
    <li>Friends: </li>
    <li><a href="https://blog.alalin.me/" target="_blank">alalin.me</a></li>
  </ul>
</div>
```
VS
```
<div class="footer-col footer-col-2">
  {%- include social.html -%}
</div>
```

I also added my DigitalOcean referral link.

```
<div class="footer-col-wrapper">
  <div class="footer-col footer-col-1">
        ...
        {%- if site.email -%}
        <li><a class="u-email" href="mailto:{{ site.email }}">{{ site.email }}</a></li>
        {%- endif -%}
    </ul>
+ <a href="https://m.do.co/c/de1b31930850" target="_blank"><img src="/images/DO_Powered_by_Badge_black.svg" ></a>
  </div>
```

You can download the official DigitalOcean logo [here](https://www.digitalocean.com/press/)

Last, commit and git push. There you have it.

![Customized Footer](/images/minima_customized_footer.png)

Our customized `_includes/footer.html` here will overide the default `footer.html` used by Minima. 

Besides, You can also customize the CSS style.

## Reference
* [https://github.com/jekyll/minima](https://github.com/jekyll/minima)
* [https://jekyll.github.io/minima/](https://jekyll.github.io/minima/)
