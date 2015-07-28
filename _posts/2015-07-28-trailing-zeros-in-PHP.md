---
layout:     post
title:      Trailing zeros in PHP
date:       2015-07-22
summary:    I've recently joined Villa-Finder, where lots of large projects where waiting for me with lots of non-optimised/dead code. Here is a little something that help me cleaned up this Symfony2 entity.
categories: symfony2
tags: symfony2 php cleanup
---

I've recently joined Villa-Finder, where lots of large projects where waiting for me with lots of non-optimised/dead code. Here is a little something that help me cleaned up this Symfony2 entity.

## The bad and the ugly

I recently found this nice piece of really non-optimised code, playing with concatenation and calculation <del>and sadness 😭</del>.

{% highlight php startinline=true %}
public function addTrailingZeros($total)
{
    $total = (float) $total * 100;
    $totalWithTrailingZeros = "";
    $totalWithTrailingZeros .= ($total / 100000000000) % 10;
    $totalWithTrailingZeros .= ($total / 10000000000) % 10;
    $totalWithTrailingZeros .= ($total / 1000000000) % 10;
    $totalWithTrailingZeros .= ($total / 100000000) % 10;
    $totalWithTrailingZeros .= ($total / 10000000) % 10;
    $totalWithTrailingZeros .= ($total / 1000000) % 10;
    $totalWithTrailingZeros .= ($total / 100000) % 10;
    $totalWithTrailingZeros .= ($total / 10000) % 10;
    $totalWithTrailingZeros .= ($total / 1000) % 10;
    $totalWithTrailingZeros .= ($total / 100) % 10;
    $totalWithTrailingZeros .= ($total / 10) % 10;
    $totalWithTrailingZeros .= ($total / 1) % 10;
    
    return $totalWithTrailingZeros;
}
{% endhighlight %}

So here, the author is trying to _left align the original number in a 12 characters chain, with trailing zeros_.

Example: `149.99` will become `000000014999`. 

## The good 

Hopefully, there is a lovely 😘 way to add trailing chars:

{% highlight php startinline=true %}
public function addTrailingZeros($total)
{
    return sprintf('%012d', $total * 100);
}
{% endhighlight %}

With the [`sprintf`](http://php.net/sprintf) function you can specify a padding in the format part: 

* `%d` is to display an integer, 
* `%12d` will __right align__ the integer in a __12 character long string__ (you can left align with `%-12d`),
* `%012d` will replace the empty chars with zeros. For other replacement, like a dot: `%'.12d`.

And there is also a dedicated PHP function to pad a string, [`str_pad`](http://php.net/str-pad). Used in my method:

{% highlight php startinline=true %}
public function addTrailingZeros($total)
{
    return str_pad($total, 12, "0", STR_PAD_LEFT);
}
{% endhighlight %}
