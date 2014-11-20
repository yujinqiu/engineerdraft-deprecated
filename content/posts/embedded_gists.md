---
title: "Embedded Gists"
description: "Perhaps it is better to embed gists than have inline markup in blog posts."
date: "2014-07-05"
categories: 
    - "tech"
    - "github"
    - "gists"
    - "hugo"
---

## Syntax Highlighting - or Embedded Gists

I was quite excited by the inline syntax highlighting that Hugo provides via the python plugin [Pygments](http://pygments.org/). But also wanted to try embedding gists 

So here is an example...

{{% highlight python %}}
def digit_sum(n):
    s = 0
    while n:   # while n is 'truthy' for an integer that means not 0
        s = s + (n % 10)   # the sum is the sum + the remainder of dividing the incoming number (n) by 10  157 % 10 = 7
        n = n / 10         # n = the integer of n / 10   int(15.7) = 15 
    return s
   
print digit_sum(157)

'''
calling digit_sum(157)
 
loop  | s   | n   | n % 10  | n / 10
------------------------------------
0     | 0   | 157 |   7     |  15
1     | 7   | 15  |   5     |  1
2     | 12  | 1   |   1     |  0
3     | 13  | 0   |         |  
'''
{{% /highlight %}}



But then thought perhaps it is better to embed gists than have inline markup in blog posts then folk can fork the code and comment there. 

So on embedding a gist as follows (but without syntax highlighting) failed...
<!--more-->

{{% highlight html %}}
<script src="https://gist.github.com/danmux/042fa69bed3791afe658.js">
</script>
{{% /highlight %}}

Putting it in another block partially worked 

{{% highlight html %}}
<p>
    <script src="https://gist.github.com/danmux/042fa69bed3791afe658.js">
    </script>
</p>
{{% /highlight %}}

I was left with an unclosed script tag.

A git of googling turned up this thread...

https://groups.google.com/forum/#!topic/hugo-discuss/3GW56aMYQF8

and this pull request ...

https://github.com/spf13/hugo/pull/305

which missed the latest release by a few days...

https://github.com/spf13/hugo/releases/tag/v0.11

So I went ahead an built the master head - from my fork (ya know just in case I can help)

and added the script tag in....

<script src="https://gist.github.com/danmux/042fa69bed3791afe658.js"></script>


### And it worked ! 

Nice work [jmitchell](https://github.com/jmitchell) and [Steve Francia](https://github.com/spf13).

So gists it is for me

