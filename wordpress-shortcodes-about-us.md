# Introduction


This is pretty garbagy so here's what's going on.

This blog post is to create 3 shortcodes that get outputted into a About Page.

[Screenshot]

We'll be creating a: 

1. Video Section Shortcode
2. Hero Section Shortcode
3. Repeatable Social Media Shortcode

## Why shortcodes instead of Blocks

The intention is that these shortcodes then become Gutenberg Blocks.

But right now, I'm stuck with trying to create Gutenberg Blocks with Vue, as they are React based. 

I'm also having a lot of trouble with [Block API Version 2](https://make.wordpress.org/core/2020/11/18/block-api-version-2/).

Almost all tutorials are before this change, and the Version 2 structure is different and all my attempts to switch to Version 2 have been a headache.

Overall, it's a crap experience and I hate my life.

## Video Section Shortcode


Within the Hero Image Block, we ask for 8 inputs.

1. 'autoplay' => (optional) start the video when the page loads
1. 'muted' => (optional) The video defaults to muted
1. 'loop' => (optional) play it again after it finishes
1. 'controls' => (optional) show controls. Defaults to true.
1. 'display' => (optional) if the video is 100% or half the size. Didn't really think this through.
1. 'preload' => (optional) How much video will download.
1. 'poster' => (optional) If there's a custom image before it plays.
1. 'source' => The only thing that is necessary. A direct link to the video.

`[shortcode-video-block]`

```html
[shortcode-video-block autoplay=true muted=true looped=true controls=true
    poster="http://media.w3.org/2010/05/sintel/poster.png"
    source="http://media.w3.org/2010/05/sintel/trailer.mp4"]
```

Some thing to notice: 
`$videoTypeCheck` will pull the file extension and determine what video type to use.


```php 
function shortcode_video_block($atts = [], $content = null) {

    $blockAtts = shortcode_atts(
        [
            'autoplay' => false,
            'muted' => false,
            'loop' => false,
            'controls' => true,
            'display' => 'wide',
            'preload' => 'metadata',
            'poster' => '',
            'source' => ''
        ], $atts);


    // booleans
    $isAutoplay = ($blockAtts['autoplay']) ? 'autoplay' : '';
    $isMuted = ($blockAtts['muted']) ? 'muted' : '';
    $isLooped = ($blockAtts['loop']) ? 'loop' : '';
    $videoControls = ($blockAtts['controls']) ? 'controls' : '';

    $display = $blockAtts['display'];
    $preload = $blockAtts['preload']; //none or metadata
    $poster = $blockAtts['poster'];
    $source = $blockAtts['source'];

    // display
    $inlineStyle = 'margin: 0 auto;';
    if ($display == 'wide') {
        $inlineStyle .= 'width: 100%;';
    } else {
        $inlineStyle = 'width: 50%;';
    };

    // get video type
    $videoType = '';

    $videoTypeCheck = substr($source, strrpos($source, '.')+1);
    switch ($videoTypeCheck) {
        case "mp4";
        case "m4v";
        case "m4p";
            $videoType = 'mp4' ;
            break;
        case "ogv";
        case "ogg";
            $videoType = 'ogg' ;
            break;
        case "webm";
            $videoType = 'webm' ;
            break;
        case "mpg";
        case "mpeg";
            $videoType = 'mpeg';
        default:
            $videoType = 'error';
    }

    if ($videoType == 'error') {
        return '<p>There is an error with your video link.</p>';
    }

    ob_start();
    ?>
    <div style="<?php $inlineStyle; ?>">
            <video
                <?php echo "$videoControls $isAutoplay $isLooped $isMuted"; ?>
                preload="<?php echo $preload; ?>"
                poster="<?php echo $poster; ?>"
                width="100%"
            >
            <source src="<?php echo $source; ?>" type="video/<?php echo $videoType; ?>">
            Sorry, your browser does not support embedded videos.
        </video>
    </div>
    <?php

    return ob_get_clean();

}
```


## Hero Image Block

Within the Hero Image Block, we ask for 7 inputs.

1. header => The text
2. subheader => (Optional) Smaller text.
3. text-color => (Optional) If you want it to be a specific color.
4. text-location => if you want the text to be top, centered, or bottom.
5. img-size => (optional) If you want the image to fit the container, or be automatically cropped.
6. img-src => A URL of where the image is located.


Default block:
`[shortcode-hero-block]`

Example:
```html
[shortcode-hero-block
    header="This is the best thing ever"
    subheader="Now you know"
    text-color="white"
    img-size="cover"
    img-src="https://blog.pixlr.com/wp-content/uploads/2019/03/stars-pattern.png"
]
```

This uses inline code to generate the block, to avoid CSS clashing.

```php 
function shortcode_hero_block($atts = [], $content = null) {

    $blockAtts = shortcode_atts(
        [
            'header' => "Replace Me",
            'subheader' => '',
            'text-color' => 'black',
            'text-location' => 'center', // top, bottom, center
            'img-height' => '400',
            'img-size' => 'contain',
            'img-src' => 'https://i.imgur.com/QsiWlp0.jpeg'
        ], $atts);

    switch($blockAtts['text-location']) {
        case "top":
            $blockAtts['text-location'] = 'flex-start';
            break;
        case "bottom":
            $blockAtts['text-location'] = 'flex-end';
            break;
        default:
            break;
    }

    $header = $blockAtts['header'];
    $subheader = $blockAtts['subheader'];

    $textColor = 'color: ' . $blockAtts['text-color'] . ';';

    $display = 'width: 100%; display: flex; flex-direction: column; padding: 2.5rem 1rem;'; //wide or normal?
    $textLocation = 'justify-content: ' . $blockAtts['text-location'] . ';';
    $imageHeight = 'height: ' . $blockAtts['img-height'] . 'px;';
    $backgroundType = 'background-size: ' . $blockAtts['img-size'] . ';'; //contain or cover
    $backgroundRepeat = 'background-repeat: no-repeat;';

    $imgSource = 'background-image: url(' . $blockAtts['img-src'] . ');';

    $inlineCSS = $display . $textLocation . $imageHeight . $backgroundType . $backgroundRepeat . $imgSource;

    ob_start();
    ?>
        <div style="<?php echo $inlineCSS ?>">
            <h2 style="<?php echo $textColor ?>"><?php echo $header; ?></h2>
            <?php if (!empty($blockAtts['subheader'])): ?>
                <h3 style="<?php echo $textColor ?>"><?php echo $subheader; ?></h3>
            <?php endif; ?>
        </div>
    <?php

    return ob_get_clean();

  }
add_shortcode('shortcode-hero-block', 'shortcode_hero_block');
```




## Social Media Block

Within the Social Media Block, we ask for 2 inputs.

1. `inner-text` => We want the name of social media handle
2. `link` => The social media link

Default block:
`[shortcode-social-media-block]`

Examples: 
```html
[shortcode-social-media-block
  inner-text="Linked in!"
  link="https://www.linkedin.com/in/foone-turing-7921a2100/"
]

[shortcode-social-media-block
  inner-text="the Facebook!"
  link="https://www.facebook.com/jornaloglobo"
]

[shortcode-social-media-block
  inner-text="the twitter!"
  link="https://twitter.com/jemyoung"
]

[shortcode-social-media-block
  inner-text="the pinterest!"
  link="https://www.pinterest.com/joannamagrath/pinterest-squirrels/"
]

[shortcode-social-media-block
  inner-text="the instagram!"
  link="https://www.instagram.com/dogsofinstagram/"
]
```

Our code will also check what type of link you included, and automatically generate a social media icon next to it from the [Dashicon library](https://developer.wordpress.org/resource/dashicons/). 

If your social media link isn't on the approved list within `$socialMediaChoices`, it'll output a error.
If it's successful, it'll output correctly.

Additionally, this shortcode is inline. 

```php
function shortcode_social_media_block($atts = [], $content = null) {

    $blockAtts = shortcode_atts(
        [
            'inner-text' => "@chriscoyier",
            'link' => 'https://twitter.com/chriscoyier',
        ], $atts);

    // check the type of social media gives you a dashicon
    $socialMediaTitle = $blockAtts['inner-text'];
    $link = $blockAtts['link'];

    $inlineStyle = 'line-height: inherit; text-decoration: none;';

    $socialMediaChoices = ['facebook', 'twitter', 'linkedin', 'pinterest', 'instagram'];

    foreach ($socialMediaChoices as $socialMedia) {
        if (strpos($link, $socialMedia)) {
            $linkType = $socialMedia;
            break;
        }
    }

    // SET ERROR MESSAGE
    if (!isset($linkType)) {
        $errorMsg = "Your social media choice doesn't exist. Try again";
    }

    // ERROR MESSAGE
    // If there's an error, return the error message.
    if (isset($errorMsg)) {
        return "<p>$errorMsg</p>";
    }

    ob_start();
    ?>
        <a href="<?php echo $link; ?>" target="_blank" rel="noopener"><span style="<?php echo $inlineStyle; ?>" class="dashicons dashicons-<?php echo $linkType; ?>"></span><?echo $socialMediaTitle; ?></a>
    <?php

    return ob_get_clean();

  }
add_shortcode('shortcode-social-media-block', 'shortcode_social_media_block');
```

## Conclusion

Everything sucks.
