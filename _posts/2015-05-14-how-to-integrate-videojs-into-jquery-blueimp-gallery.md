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

{% highlight coffeescript %}
((factory) ->
  "use strict"
  if typeof define is "function" and define.amd

    # Register as an anonymous AMD module:
    define [ "./blueimp-helper", "./blueimp-gallery" ], factory
  else

    # Browser globals:
    factory window.blueimp.helper or window.jQuery, window.blueimp.Gallery
) ($, Gallery) ->
  "use strict"

  handleSlide = Gallery::handleSlide

  $.extend Gallery::,
    handleSlide: (index) ->
      handleSlide.call this, index

      for video in (@playingVideos or [])
        video.pause() if video?

    generateGuid: (->
      s4 = ->
        Math.floor((1 + Math.random()) * 0x10000).toString(16).substring 1
      ->
        s4() + s4() + "-" + s4() + "-" + s4() + "-" + s4() + "-" + s4() + s4() + s4()
    )()

    buildTemplate: (obj, wrapper_id, video_id) ->
      poster  = $(obj).data('poster')
      sources = $(obj).data('sources').reverse()
      title   = $(obj).attr 'title'

      $wrapper_tag = $('<div>')
        .addClass 'video-js-box video-content'
        .attr 'title', title
        .attr 'id',    wrapper_id

      $poster_tag = $('<img>')
        .addClass 'toggle poster-video'
        .attr 'src', poster

      $video_tag = $('<video>')
        .addClass 'video-js vjs-default-skin'
        .prop 'controls', true
        .prop 'autoplay', false
        .attr 'preload',  'auto'
        .attr 'autobuffer', 'autobuffer'
        .attr 'poster',   poster
        .attr 'src',      $(obj).attr('href')
        .attr 'id',       video_id
        .attr 'width',    '70%'
        .attr 'height',   '85%'
        .css  { display:  'none' }

      $play_tag = $('<a>')
        .addClass 'play-video'

      $.each sources, (data) ->
        $source = $('<source>')
          .attr 'src',  @['href']
          .attr 'type', @['type']

        $video_tag.append $source

      $wrapper_tag
        .append $poster_tag
        .append $video_tag
        .append $play_tag

      $wrapper_tag[0]

    videoFactory: (obj, callback, videoInterface) ->
      wrapper_id = @generateGuid()
      video_id   = @generateGuid()

      $template = @buildTemplate(obj, wrapper_id, video_id)

      @setTimeout callback, [
        target: $template
        type:   'load'
      ]

      $(document).on "click", "##{wrapper_id} .play-video", (e) =>
        $("##{wrapper_id} .play-video, ##{wrapper_id} .poster-video")
          .css display: 'none'

        $("##{video_id}")
          .css display: 'block'

        @playingVideos ||= []
        @playingVideos[@index] = videojs video_id, {}, ->
          @play()

      return $template

  Gallery
{% endhighlight %}


##### Stylesheets:

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
    <a class="thumbnail" data-gallery="false" data-poster="/resource/video/large_frame/video_def.png" data-sources="[{&quot;href&quot;:&quot;/resource/video/webm/video_def.webm&quot;,&quot;type&quot;:&quot;video/webm&quot;},{&quot;href&quot;:&quot;/resource/video/mp4/video_def.mp4&quot;,&quot;type&quot;:&quot;video/mp4&quot;},{&quot;href&quot;:&quot;/resource/video/ogv/video_def.ogv&quot;,&quot;type&quot;:&quot;video/ogg&quot;}]" href="/resource/video/webm/video_def.webm" title="vamos" type="video/webm">
      <img alt="video def" src="/resource/video/thumb/video_def.png">
    </a>
  </div>
  <div class="col-md-6">
    <a class="thumbnail" data-gallery="false" data-poster="/resource/video/large_frame/video_spacetestsmall_512kb.png" data-sources="[{&quot;href&quot;:&quot;/resource/video/webm/video_spacetestsmall_512kb.webm&quot;,&quot;type&quot;:&quot;video/webm&quot;},{&quot;href&quot;:&quot;/resource/video/mp4/video_spacetestsmall_512kb.mp4&quot;,&quot;type&quot;:&quot;video/mp4&quot;},{&quot;href&quot;:&quot;/resource/video/ogv/video_spacetestsmall_512kb.ogv&quot;,&quot;type&quot;:&quot;video/ogg&quot;}]" href="/resource/video/webm/video_spacetestsmall_512kb.webm" title="Lorem ipsum..." type="video/webm">
      <img alt="Video spacetestsmall 512kb" src="/resource/video/thumb/video_spacetestsmall_512kb.png">
    </a>
  </div>
</div>
{% endhighlight %}
