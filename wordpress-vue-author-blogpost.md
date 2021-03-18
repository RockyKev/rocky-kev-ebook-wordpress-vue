# The Title

It's a really good time to be a Frontend Javascript developer, with frameworks like React, Vue, Svelte, etc managing a lot of the heavy lifting. WordPress is a popular CMS in web development, [powering 39.5% of websites, and 14.7% of top websites.](https://kinsta.com/blog/wordpress-statistics/#usage-statistics) but much of it is written in PHP. 

This post will show you how to create a Vue App inside WordPress.

## Introduction
I used Jake Paris' [Using a Vue app within a WordPress Plugin](https://jake.paris/blog/2020/06/23/using-a-vue-app-within-a-wordpress-plugin/) as a kickoff point for this tutorial. If you're confused by anything, feel free to reference his post for more clarity. 

I wrote this post to add a bit of a 'Explain like i'm 5' to his post. This is super helpful if you're a Javascript/Vue Developer, who wants to bring your skillset to WordPress. Overall, if you can build a Vue App, you can then auto-magically shove it into WordPress. 

Sweet, yeah?

Note: There's a bunch of different ways to do this. Lisa Armstrong's [dev.to post](https://dev.to/workingwebsites/using-vue-in-wordpress-1b9l) is pretty good. And Evan Agee's [WP Vue plugin Starter](https://github.com/EvanAgee/vuejs-wordpress-plugin-starter) is also really good. 


## How does this work?

1. We are building a WordPress plugin in PHP. The plugin sets up JS dependencies and outputs our Vue App.
2. We are generating a Vue App inside the plugin. It'll be an Author app.
4. That Vue App gets mounted into a html element, that was generated in step 1. 

In other words - if you can build it in Vue, you can make it appear as a WordPress shortcode.

## Process: 

If you're having trouble, the final code is located in [this Repo](https://github.com/RockyKev/wordpress-vue-author). 
I'll be using `npm`. 

We'll run through everything from setting up your WordPress environment/WordPress plugin, to creating the Vue plugin.

Let's begin: 

Step 1: Create a WordPress plugin. 
Step 2: Create a Vue App. 
Step 3: Get Vue working inside of our WordPress plugin shortcode

## Step 1: Create a WordPress Plugin

1. Set up a WordPress environment. 

There are a lot of modern tools to deploying WordPress these days. For the most quickest deployment, I love [Local by Flywheel](https://localwp.com/). It's one-click setup. It assumes you want the latest version of WordPress.

[screenshot]

For those who want more control, I also highly recommend [Devilbox](http://devilbox.org/). There's also [WordPress for Docker](https://hub.docker.com/_/wordpress). Just stay away from the MAMPP/XAMPP/WAMP because this isn't 2015 and life is better now.

2. Create a new folder named `vue-author-generator` inside `wp-contents/plugins/`

```
// In the wp-contents folder
// Note - we'll generate the app folder later.

/plugins
└───vue-author-generator/
   │    
   └───vue-author-generator.php

```

Add the following content in there. 
NOTE: Much of this was yoinked from: https://wppb.me/

```php 
// inside vue-author-generator.php
<?php

/**
 * The plugin bootstrap file
 *
 * This file is read by WordPress to generate the plugin information in the plugin
 * admin area. This file also includes all of the dependencies used by the plugin,
 * registers the activation and deactivation functions, and defines a function
 * that starts the plugin.
 *
 * @link              #
 * @since             1.0.0
 * @package           Vue Author Generator
 *
 * @wordpress-plugin
 * Plugin Name:       Vue Author Generator
 * Plugin URI:        #
 * Description:       This creates a Vue Author that passes PHP data to a plugin.
 * Version:           1.0.0
 * Author:            Your Name Here
 * Author URI:        #
 * Text Domain:       author-generator-vue
 */

// If this file is called directly, abort.
if (!defined('WPINC')) {
	die;
}

/**
 * Currently plugin version.
 */
define('AUTHOR_GENERATOR_VUE', '1.0.0');

// Invoke Shortcode
function vue_author_generator() {
    return "<p>I love pressure, I eat it for breakfast!</p>"; 
}

add_shortcode('generate-author-vue', 'vue_author_generator');
```

What we're doing here is creating a WordPress shortcode. When we use it in a post/page within WordPress, it'll output the text. 
[Here's a complete guide to WordPress shortcodes](https://www.smashingmagazine.com/2012/05/wordpress-shortcodes-complete-guide/)


3. Set up the WordPress side

We have our plugin set up. Now we need to Activate it.

Visit your WordPress backend. 
On the sidebar, click Plugins > Installed Plugins
https://i.imgur.com/QJshSpp.png


4. Do a test run to make sure this plugin works.

Now visit a page, add your shortcode in there. 
If it's done correctly, then congrats -- you made a plugin! (A very limited one)

BEFORE:
https://i.imgur.com/SgdgZVA.png

AFTER:
[SCREENSHOT HERE]

If is isn't working: 
1. Check that the PHP code is the same. 
2. Check that the shortcode inside WordPress is correct. 
3. Check that your plugin is ACTIVATED in WordPress.

## Step 2:  Create a Vue App

With our WordPress plugin working, we're now going to generate the Vue side of things.
How this works is that inside our WordPress plugin is a Vue app. The WordPress plugin is grabbing the compiled JS code and displaying it through the shortcode. 

1. Install the [vue-cli](https://cli.vuejs.org/guide/) with `npm install -g @vue/cli`

This will allow us to use the Vue command in our terminal.

2. Navigate to your WordPress Plugin. 

`/wp-contents/plugins/`

3. Generate a new Vue project called `app` 

```
plugins
└───vue-author-generator/
   │    
   └───app                          ## Your Vue folder
   │    │   
   │    └───dist                    ## When you build files
   │    └───node_modules                       
   │    └───public                          
   │    └───src                     ## Where you modify files   
   │    └───package.json
   │         ...
   │    
   └───vue-author-generator.php
```


`vue create app`

This will generate your Vue App. 

Here's my settings: 
? Please pick a preset: Manually select features
? Check the features needed for your project: Choose Vue version, Babel, CSS Pre-processors, Linter
? Choose a version of Vue.js that you want to start the project with 3.x (Preview)
? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): Sass/SCSS (with dart-sass)
...
The other choices are personal preferences. 


It should take a few minutes to install.



4. Add our new content in there.

First, replace the App.vue with this:

```vue
// We are inside `vue-author-generator/app/src/App.vue`

<template>
  <div>
    <AuthorBio />
  </div>
</template>

<script>
import AuthorBio from "./components/AuthorBio.vue";

export default {
  name: "App",
  components: {
    AuthorBio,
  },
};
</script>

<style>
</style>
```

Second, create a new compentent called AuthorBio
```vue
// We are inside `vue-author-generator/app/src/components/Author.vue`

<template>
  <div class="authorbox">
    <div class="profile">
      <img src="https://www.placecage.com/c/400/400" />
      <div class="social">
        <!-- ADDITIONAL: check if the value exists before outputting --> 
        <a href="#" target="_blank">FB</a>|
        <a href="#" target="_blank">IN</a>|
        <a href="#" target="_blank">TW</a>| 
        <a href="#" target="_blank">WEBSITE</a>
      </div>
    </div>

    <div class="description">
      <div class="name">
        Dr. Stanley Goodspeed
      </div>
      <div class="content">
       It's a Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
      </div>
    </div>
  </div>
</template>

<template>
  <div class="authorbox">
    <div class="profile">
      <img src="https://www.placecage.com/c/400/400" />
      <div class="social">
        <a href="#" target="_blank">FB</a>|
        <a href="#" target="_blank">IN</a>|
        <a href="#" target="_blank">TW</a>| 
        <a href="#" target="_blank">WEBSITE</a>
      </div>
    </div>

    <div class="description">
      <div class="name">
        Dr. Stanley Goodspeed
      </div>
      <div class="content">
       It's a Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
      </div>
    </div>
  </div>
</template>




<style lang="scss" scoped>
.social {
  text-align: center;
  display: flex;
  justify-content: space-between;
  width: 75%;
}

.authorbox {
  display: flex;
  justify-content: center;
  align-items: center;

  .profile {
    flex: 0 1 25%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .description {
    text-align: left;
    flex: 0 1 75%;
  }
}

.profile {
  img {
    width: 300px; 
    border-radius: 50%;
  }
}

.description {
  margin-left: 2rem;

  .name {
    font-size: 2.5rem;
  }
  .content {
    line-height: 1.1;
  }
}
</style>
```

5. Run the the app.

`cd app`
`vue run serve` 
It'll open the Vue project in [http://localhost:8080/](http://localhost:8080/).

If everything worked, you should see your author page.

[SCREENSHOT]https://i.imgur.com/NJ7RdXY.png


## Step 3: Get Vue working inside of our WordPress plugin shortcode

This is the complicated part. 
We now need to pass the Vue data over to PHP. PHP will then output the vue App inside the shortcode. 

1. Preparing the Vue side 

First, `npm run build` To create a `dist` folder. 

Next, we'll need to create a [vue.config.js](https://cli.vuejs.org/config/#global-cli-config) inside the `app` folder. 

By default, Vue appends a hash to files in your `dist` folder. We need to disable that to make it work with PHP. We'll also need to modify some Vue Webpack configs, which we'll use the [chainWebpack](and chainWebpack (https://cli.vuejs.org/guide/webpack.html#modifying-options-of-a-plugin)) method.

```js

// this is located in wordpress-vue-author/app/vue.config.js

module.exports = {
    filenameHashing: false,
    publicPath: "./",
    outputDir: "dist",
    chainWebpack: (config) => {
      config.plugins.delete("html");
      config.plugins.delete("preload");
      config.plugins.delete("prefetch");
      config.module
        .rule("images")
        .use("url-loader")
        .tap((options) => {
          options.name = "/img/[name].[ext]";
          options.publicPath =
            "../wp-content/plugins/vue-author-generator/app/dist/";
          return options;
        });
    },
    devServer: {
      writeToDisk: true,
      hot: false,
    },
    productionSourceMap: false,
  };
```
*This was borrowed from Jake Paris' [Using a Vue app within a WordPress Plugin](https://jake.paris/blog/2020/06/23/using-a-vue-app-within-a-wordpress-plugin/) post.*


2. Preparing wordpress side

Now open up `vue-author-generator.php`
Your Function should now look like this.

```php
<!-- We are inside wordpress-vue-author/vue-author-generator.php -->

function vue_author_generator() {
    // get Vue libs 
	wp_register_script('vue-app-vendors',  plugins_url('app/dist/js/chunk-vendors.js', __FILE__), array(), '1.0.0');
	wp_register_script('my-vue-app', plugins_url('app/dist/js/app.js', __FILE__), array('vue-app-vendors'), '1.0.0');

	// pass the scripts to WP
	wp_enqueue_script('vue-app-vendors');
	wp_enqueue_script('my-vue-app');
	wp_enqueue_style('my-vue-app',  plugins_url('app/dist/css/app.css', __FILE__),   array(),  '1.0.0');

    return '<div id="app"></div>';
}
```

What's happening is that:

First, [registering and enqueuing js/css files the WordPress way](https://www.wpbeginner.com/wp-tutorials/how-to-properly-add-javascripts-and-styles-in-wordpress/).

Second, getting the Vue dependencies, the App's CSS and your Vue App code.

Finally, that code is going to be outputted in that `<div id="app"></div>` element.

3. If everything was set up correctly, your Vue app should now appear in WordPress. Woohoo!

[SCREENSHOT] https://i.imgur.com/GwhBOI6.png

## Conclusion

This is the first step. You can now make a fully-realized Vue App inside WordPress, and have it called by shortcode. You can even migrate it out of the shortcode into a [WordPress Page template](https://www.smashingmagazine.com/2015/06/wordpress-custom-page-templates/). And additionally, in a future blog post, I'll explain how to pass data from the WordPress CMS, into your Vue Plugin, and have it outputted within Vue. 

