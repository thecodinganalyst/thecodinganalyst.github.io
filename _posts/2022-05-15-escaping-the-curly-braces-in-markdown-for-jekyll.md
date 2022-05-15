---
title: "Escaping the {{ }} in markdown for Jekyll"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Jekyll
  - Markdown
  - Liquid Templating Language
---

I just realised that in [one of my previous posts](https://thecodinganalyst.github.io/knowledgebase/Basic-guide-to-Semantic-Release/), the code I had for `{% raw %}${{ secrets.GH_TOKEN }}{% endraw %}` became just `$`. 

After a bit of [research](https://stackoverflow.com/questions/39627452/how-to-use-with-markdown), I found out that it is due to the fact that `{% raw %}{{  }}{% endraw %}` is used for variable interpolation in [liquid](https://github.com/Shopify/liquid), and [liquid is used by Jekyll](https://jekyllrb.com/docs/liquid/#:~:text=Jekyll%20uses%20the%20Liquid%20templating,e.g.%20%7B%25%20if%20statement%20%25%7D%20.). 

So every time I need to use the `{% raw %}{{  }}{% endraw %}` in my code examples, I need to add the `{``%`` raw ``%``}` and `{``%`` endraw ``%``}` around it to escape the values. 