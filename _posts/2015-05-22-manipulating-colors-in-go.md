---
layout: post
title:  "Manipulating Colors in Go"
date:   2015-05-22 01:07:47
categories: go golang colors 
---

This took me a while to figure out so I figured I better document it. 

I'm working on a project that requires that I get the average color for a section of an image. This was the function I created to do that:
{% highlight go linenos %}
func AverageImageColor(i image.Image) color.Color {
	var r, g, b uint32

	bounds := i.Bounds()

	for y := bounds.Min.Y; y < bounds.Max.Y; y++ {
		for x := bounds.Min.X; x < bounds.Max.X; x++ {
			pr, pg, pb, _ := i.At(x, y).RGBA()

			r += pr
			g += pg
			b += pb
		}
	}

	d := uint32(bounds.Dy() * bounds.Dx())

	r /= d
	g /= d
	b /= d

	return color.NRGBA{uint8(r), uint8(g), uint8(b), 255}
}
{% endhighlight %}

Unfortunately this wasn't working and I was instead getting random colors returned. It certainly wasn't the average color I was hoping for. 

The problem lay in the last line. When you call the `color.RGBA()` method it takes a `uint8` and converts it to a `uint32`. The reasoning is that it avoids overflow if you are trying to do some kind of calculation on your RGB values. To do that conversion it first has to change the 8 bit value to a 16 bit value then from there it drops the 16 bit value into a 32 bit integer container. To convert from 8 to 16 bits it multiplies the RGB values by 0x101 meaning if you ever want to convert back down to 8 bit you need to divide by that same value. 

So the return statement of my AverageImageColor function should look as follows:

{% highlight go %}

return color.NRGBA{uint8(r / 0x101), uint8(g / 0x101), uint8(b / 0x101), 255}

{% endhighlight %}

Here's the difference that makes in the image:

![Bad output]({{ site.url }}/assets/2015-05-22_bad_out.png)
![Good output]({{ site.url }}/assets/2015-05-22_good_out.png)