---
layout: post
title: 带缩略图的图片轮播插件
location: 北京
category: tech
tags: [jquery, slider, javascript, css]
css: flexslider.css
---
<script src="{{ site.baseurl }}/js/jquery-1.11.1.min.js"></script>
<script src="{{ site.baseurl }}/js/jquery.flexslider.js"></script>
<script src="{{ site.baseurl }}/js/modernizr.js"></script>
前些天做一个网页，需要用到图片轮播的功能。想着这个东西应该有现成的东西，于是找了一下。大致有两种，Flash的和JavaScript实现的。由于Flash没有涉猎，不懂这方面，于是找了一下JavaScript实现的图片轮播组件。

简单查看了一些图片轮播组件的效果之后，选择了使用[FlexSlider](http://flexslider.woothemes.com/thumbnail-controlnav.html)这个组件。简单易用，满足要求。缩略图和图片轮播等功能很好地实现了，但是也有不足之处，就是不能添加图片说明。既然没有提供，那就自己手动添加一个。当然添加之前写找了一下有没有人已经做过这个事了，搜寻一番之后发现要么是实现的过于复杂，要么就没有。最后决定还是自己加吧，应该不是很困难。折腾了一会之后，最后还是弄出来了，主要的一个问题是图片说明的背景有部分是相互重叠的，设置了透明度之后，重叠的部分后面的会和前面的叠加，造成看到的背景色颜色不一致，最后的解决办法是设置了图片说明的背景条的长度，确保不会重叠，具体参见[flexslider.css]({{ site.baseurl }}/css/flexslider.css)这个文件。做这个工具的作者大概是个吃货，Demo图是看起来非常好吃的蛋糕，看着就很过瘾。前后两个效果分别如下：

<section class="slider">
    <div class="flexslider">
        <ul class="slides">
            <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_cheesecake_brownie.jpg">
  	    	<img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_cheesecake_brownie.jpg" />
  	    </li>
  	    <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_lemon.jpg">
  	    	<img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_lemon.jpg" />
  	    </li>
  	    <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_donut.jpg">
  	    	<img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_donut.jpg" />
  	    </li>
  	    <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_caramel.jpg">
  	        <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_caramel.jpg" />
  	    </li>
        </ul>
    </div>
</section>

<section class="slider">
    <div class="flexslider">
        <ul class="slides">
            <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_cheesecake_brownie.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_cheesecake_brownie.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是布朗尼芝士蛋糕 </a>
		</div>
  	    </li>
  	    <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_lemon.jpg">
  	    	<img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_lemon.jpg" />
                <div class="flex-caption">
	             <a href="#">这个是柠檬 </a>
		</div>
  	    	</li>
  	    	<li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_donut.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_donut.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是甜甜圈</a>
		</div>
  	    	</li>
  	    	<li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_caramel.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_caramel.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是焦糖</a>
		</div>
  	    </li>
        </ul>
    </div>
</section>

这个插件还是很好用的，样式也非常漂亮。但是做好之后，却被告知需要把缩略图放在图片的右边。orz...既然说要改，那也只好改了。于是又费了一番功夫把缩略图放到右边，最终的效果图如下：

<section class="slider" id="right-slider">
    <div class="flexslider">
        <ul class="slides">
            <li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_cheesecake_brownie.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_cheesecake_brownie.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是布朗尼芝士蛋糕 </a>
		</div>
  	    	</li>
  	    	<li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_lemon.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_lemon.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是柠檬 </a>
		</div>
  	    	</li>
  	    	<li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_donut.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_donut.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是甜甜圈</a>
		</div>
  	    	</li>
  	    	<li data-thumb="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_caramel.jpg">
  	    	    <img src="{{ site.baseurl }}/images/flexslider/kitchen_adventurer_caramel.jpg" />
                <div class="flex-caption">
		    <a href="#">这个是焦糖</a>
		</div>
  	    </li>
        </ul>
    </div>
</section>

<script type="text/javascript">
    $(window).load(function(){
      $('.flexslider').flexslider({
        animation: "slide",
        controlNav: "thumbnails",
        start: function(slider){
          $('body').removeClass('loading');
        }
      });
    $('#right-slider .flexslider').flexslider({
        animation: "slide",
        controlNav: "thumbnails",
        itemWidth: 690,
        start: function(slider){
          $('body').removeClass('loading');
        }
      });
    });
</script>

这样就可以利用FlexSlider变换出各种你所需要的样式，可以满足大部分需求。当然，最后的这个效果，图片之间的切换还是从左至右，最一致的做法应该是从上到下。但是直接修改FlexSlider的参数好像还不能直接达到要求，还需要调整CSS样式。还有一个取巧的方式，就是把图片切换的效果改成淡入和淡出，这样就不存在方向性问题了。

当然，如果你没有强迫症，目前的这种效果也完全OK了。

The end.
