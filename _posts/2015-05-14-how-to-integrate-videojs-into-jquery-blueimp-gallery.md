---
layout: post

title: 'How to integrate VideoJS into jQuery Blueimp Gallery'
subtitle: 'Create a custom factory to integrate VideoJS into Blueimp Gallery'
cover_image: videojs_blueimp_gallery.jpg

excerpt: "Create a custom factory to integrate VideoJS into Blueimp Gallery"

author:
  name: Andrea Dal Ponte
  twitter: dalpo
  gplus: 100687498195339762535
  bio: Full stack web developer
  image: avatar.png
---

Blueimp Gallery is a touch-enabled, responsive and customizable image & video gallery, carousel and lightbox, optimized for both mobile and desktop web browsers. It enable nativaly the support for Youtube, Vimeo and HTML5 video support.

You could extend the support to the HTML5 player VideoJS with this simply factory.


##### Javascripts:

{% highlight javascript %}
(function(factory) {
  "use strict";

  if (typeof define === "function" && define.amd) {
    return define(["./blueimp-helper", "./blueimp-gallery"], factory);
  } else {
    return factory(window.blueimp.helper || window.jQuery, window.blueimp.Gallery);
  }
})(function($, Gallery) {
  "use strict";

  var handleSlide = Gallery.prototype.handleSlide;

  $.extend(Gallery.prototype, {
    handleSlide: function(index) {
      var i, len, ref, results, video;
      handleSlide.call(this, index);
      ref = this.playingVideos || [];
      results = [];
      for (i = 0, len = ref.length; i < len; i++) {
        video = ref[i];
        if (video != null) {
          results.push(video.pause());
        } else {
          results.push(void 0);
        }
      }
      return results;
    },

    generateGuid: (function() {
      var s4 = function() {
        return Math.floor((1 + Math.random()) * 0x10000).toString(16).substring(1);
      };

      return function() {
        return s4() + s4() + "-" + s4() + "-" + s4() + "-" + s4() + "-" + s4() + s4() + s4();
      };
    })(),

    buildTemplate: function(obj, wrapper_id, video_id) {
      var $play_tag, $poster_tag, $video_tag, $wrapper_tag, poster, sources, title;

      poster = $(obj).data('poster');
      sources = $(obj).data('sources').reverse();
      title = $(obj).attr('title');
      $wrapper_tag = $('<div>')
        .addClass('video-js-box video-content')
        .attr('title', title)
        .attr('id', wrapper_id);
      $poster_tag = $('<img>')
        .addClass('toggle poster-video')
        .attr('src', poster);
      $video_tag = $('<video>')
        .addClass('video-js vjs-default-skin')
        .prop('controls', true)
        .prop('autoplay', false)
        .attr('preload', 'auto')
        .attr('autobuffer', 'autobuffer')
        .attr('poster', poster)
        .attr('src', $(obj).attr('href'))
        .attr('id', video_id)
        .attr('width', '70%')
        .attr('height', '85%').css({
          display: 'none'
        });
      $play_tag = $('<a>')
        .addClass('play-video');

      $.each(sources, function(data) {
        var $source;
        $source = $('<source>').attr('src', this['href']).attr('type', this['type']);
        return $video_tag.append($source);
      });

      $wrapper_tag.append($poster_tag).append($video_tag).append($play_tag);

      return $wrapper_tag[0];
    },

    videoFactory: function(obj, callback, videoInterface) {
      var $template, video_id, wrapper_id;
      wrapper_id = this.generateGuid();
      video_id = this.generateGuid();
      $template = this.buildTemplate(obj, wrapper_id, video_id);
      this.setTimeout(callback, [
        {
          target: $template,
          type: 'load'
        }
      ]);

      $(document).on("click", "#" + wrapper_id + " .play-video", (function(_this) {
        return function(e) {
          $("#" + wrapper_id + " .play-video, #" + wrapper_id + " .poster-video").css({
            display: 'none'
          });
          $("#" + video_id).css({
            display: 'block'
          });
          _this.playingVideos || (_this.playingVideos = []);
          return _this.playingVideos[_this.index] = videojs(video_id, {}, function() {
            return this.play();
          });
        };
      })(this));

      return $template;
    }
  });

  return Gallery;
});
{% endhighlight %}


##### Stylesheets (Sass with Compass mixins):

{% highlight sass %}
@import compass/css3/transform

.blueimp-gallery
  .video-js-box.video-content
    +transform-style(preserve-3d)

    .video-js.vjs-default-skin
      +translate(-50%, -50%)
      position: absolute
      left: 50%
      top:  50%

      max-width:  100%
      max-height: 100%

      .vjs-big-play-button
        display: none

    .video-js.vjs-fullscreen
      +translate(0%, 0%)
      left: 0%
      top:  0%


  & > .slides > .slide > .video-content > video,
  & > .slides > .slide > .video-content > img
    +backface-visibility(visible)
{% endhighlight %}


##### Sample markup:

{% highlight html %}
<div class="row">
  <div class="col-md-6">
    <a class="thumbnail"
      data-gallery="false"
      data-poster="/resource/video/large_frame/video_def.png"
      data-sources="[{&quot;href&quot;:&quot;/resource/video/webm/video_def.webm&quot;,&quot;type&quot;:&quot;video/webm&quot;},{&quot;href&quot;:&quot;/resource/video/mp4/video_def.mp4&quot;,&quot;type&quot;:&quot;video/mp4&quot;},{&quot;href&quot;:&quot;/resource/video/ogv/video_def.ogv&quot;,&quot;type&quot;:&quot;video/ogg&quot;}]"
      href="/resource/video/webm/video_def.webm"
      title="Sample title" type="video/webm">
      <img alt="video def" src="/resource/video/thumb/video_def.png">
    </a>
  </div>

  <div class="col-md-6">
    <a class="thumbnail"
      data-gallery="false"
      data-poster="/resource/video/large_frame/video_spacetestsmall_512kb.png"
      data-sources="[{&quot;href&quot;:&quot;/resource/video/webm/video_spacetestsmall_512kb.webm&quot;,&quot;type&quot;:&quot;video/webm&quot;},{&quot;href&quot;:&quot;/resource/video/mp4/video_spacetestsmall_512kb.mp4&quot;,&quot;type&quot;:&quot;video/mp4&quot;},{&quot;href&quot;:&quot;/resource/video/ogv/video_spacetestsmall_512kb.ogv&quot;,&quot;type&quot;:&quot;video/ogg&quot;}]"
      href="/resource/video/webm/video_spacetestsmall_512kb.webm"
      title="Lorem ipsum..." type="video/webm">
      <img alt="Video spacetestsmall 512kb" src="/resource/video/thumb/video_spacetestsmall_512kb.png">
    </a>
  </div>
</div>
{% endhighlight %}
