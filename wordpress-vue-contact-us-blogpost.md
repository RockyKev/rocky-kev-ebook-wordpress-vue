# TITLE

## Introduction

## What we are doing:

We're covering how to send data from PHP to Vue, which then sends back to PHP.

----------
Big picture-wise -- 
We are creating a WordPress Vue plugin 
that takes data from PHP, apply it into the Vue App,
and send a ajax request to PHP, which will continue doing something with that data.


Best example of that?
A contact us form using SendGrid!

So the little picture --
1. We'll create a WordPress Plugin Settings Page that will save our Sendgrid API keys and settings.
2. That data will be sent to our Contact Us Form in Vue.
3. When we submit the Contact Us Form, it will get sent to a PHP file that fires off the SendGrid API, and sending the email.



## Process:

TODO: explain the steps
The focus is on PHP code. I'm assuming you are already proficient at Vue.

## STEP BY STEP:

Step 1: Get your sendgrid API crednetials ready
Step 2: Download my repo with a pre-built Vue Contact Us form.
Step 3: Quickly understanding the Vue Contact Us form
Step 4: Generate our Vue App as a shortcode
Step 5: Create a WP Settings Page
Step 6: Create the page where the Contact Us form takes in

### Step 1: Get your sendgrid API crednetials ready
You must have a SendGrid account to get a API.
Sendgrid has a free version for developers.

Setting up SendGrid
![](https://i.imgur.com/FF7yiHh.png)
![](https://i.imgur.com/yqx9ko0.png)

----
I didn't use wpmail because it doesn't give you error codes. 
SO you know never if it did not send or not.


### STEP 2: Download my repo with a pre-built Vue Contact Us form.
 
```
.
└── wordpress-vue-contact-us/
    ├── app/          
    ├── sendgrid-php/
    ├── vue-contact-us.php
    ├── vue-contact-us-submit.php
    └── vue-contact-us-settings-admin.php
```

Let's break those down:

`app/` folder
The Vue App that's pre-built with a Contact Us.

[screenshot]

The Contact Us form has a few different input field types. It's also broken into components for usability. 
* Text field
* TextArea field
* Radio Button
* Checkbox

There's a few custom commands that lets you build the Vue app within the root folder. 

```
 "scripts": {
    "postinstall": "npm install --prefix ./app",
    "build:app": "npm run build --prefix ./app",
    "serve:app": "npm run serve --prefix ./app"
  },
```


From within the root folder, you can `npm run postinstall`, which is a custom script command that will enter the `app` folder, and `npm install` the vue dependencies.

To build the project, you can `npm run build:app`. To run the code in dev mode, use `npm run serve:app`

The `sendgrid-php/` folder

I also downloaded the latest version of this writing of the SendGrid PHP API Library (7.9.2).
https://github.com/sendgrid/sendgrid-php


By default, Wordpress by default uses [wp_mail()](https://developer.wordpress.org/reference/functions/wp_mail/), which is a wrapper around [phpmailer()](https://github.com/PHPMailer/PHPMailer). 

There's are many of conveniences that the sendgrid-php packages has, like excellent documentation and providing error codes, things that require a lot more work if we use the wp_mailer() code.


Finally, the PHP Files: 

`vue-contact-us.php`
A blank PHP file for our WP plugin. When your WP plugin gets activated, this is the entrypoint.
We also summon the contact us settings page here.
It also contains PHP data that will pass to our Vue App.

`vue-contact-us-settings-admin.php`
This file will hold the settings page WP Admin.

`vue-contact-us-submit.php`
This file takes the Vue Contact Us submission data.
It then sanitizes the input, and sends it to the SEndGrid API, to fire off the email.
If the submission fails, it'll send a error request back.


### Step 3: Quickly understanding the Vue Contact Us form

Let's look at the Contact Us form.

[screenshot]

Here's how it looks. 

TODO: 
Talk about the debug option that you include.
How the JSON file will look
How the API call looks like. 
Some of the methods.
  like how it validates
  how it checks that all inputs are filled. 



### Step 4: Generate our Vue App as a shortcode

Let's do the easy stuff first. 

When your plugin gets activated, we create a WP Settings Page.0
```
include_once('vue-contact-us-settings-admin.php');
```

Next, we'll create a function to call our Vue App.

```
function vue_contact_us() {
    // get Vue libs
	wp_register_script('vue-app-vendors',  plugins_url('app/dist/js/chunk-vendors.js', __FILE__), array(), '1.0.0');
	wp_register_script('my-vue-app', plugins_url('app/dist/js/app.js', __FILE__), array('vue-app-vendors'), '1.0.0');

	// pass the scripts to WP
	wp_enqueue_script('vue-app-vendors');
	wp_enqueue_script('my-vue-app');
    wp_enqueue_style('my-vue-app',  plugins_url('app/dist/css/app.css', __FILE__),   array(),  '1.0.0');

	// pass plugin directory
	$wp_plugin_uri = plugin_dir_url( __FILE__ );

	wp_localize_script('my-vue-app', 'wpPluginUri', $wp_plugin_uri);

    return '<div id="app"></div>';
}

add_shortcode('contact-us-vue', 'vue_contact_us');

```

Notice this line?
```
	// pass plugin directory
	$wp_plugin_uri = plugin_dir_url( __FILE__ );

	wp_localize_script('my-vue-app', 'wpPluginUri', $wp_plugin_uri);
```

Using this, we're passing the url directory so within our Contact Us Vue App, it'll take the `wpPluginUri` data, and use it as the endpoint that the form gets submitted to.




### Step 4: Create a WP Settings Page

Let's make the WP Settings page.

It'll look like this:

![](https://i.imgur.com/IKP0KI5.png)


(I prefer this method of generating menus, which I borrowed from [Building a Better WordPress Theme with the Settings API](https://ieg.wnet.org/2015/08/build-a-better-wordpress-theme-with-the-settings-api/#:~:text=add_option%20is%20used%20to%20declare,the%20values%20of%20the%20fields.) rather than the class-based approach, which I did in [BLOG POST 2](BLOG POST 2 DIRECT LINK TO THAT STEP)

First, let's add the menu item to the WordPress Sidebar.

```php
function vue_contactus_add_theme_menu_item() {
    add_menu_page("Vue Contact Us Settings", "Vue Contact Us", "manage_options", "vue-contactus-settings", "vue_contactus_options_page", null, 99);
}
add_action("admin_menu", "vue_contactus_add_theme_menu_item");
```

Next, Let's Register all of the settings for the settings page

We need 3 options.

A input for a `from email`
A input for a `from name`
And finally, a input to stash your sendgrid api key. 

```php 
function vue_contactus_register_settings() {
    //Declare all options and if applicable, the default information
    add_option('vue_contactus_from_email', '');
    add_option('vue_contactus_from_name', '');
    add_option('vue_contactus_sendgrid_key', '');

    //Register all previously declared settings
    register_setting('vue_contactus_options', 'vue_contactus_from_email');
    register_setting('vue_contactus_options', 'vue_contactus_from_name');
    register_setting('vue_contactus_options', 'vue_contactus_sendgrid_key');
}

add_action('admin_init', 'vue_contactus_register_settings');
```

Now to create the Build the theme options page

```php 
function vue_contactus_options_page() {
?>
    <div class="wrap">
        <div>
            <h1>Theme Options</h1>
            <form method="post" action="options.php">
                <?php
                //do this before building the inputs for the settings
                settings_fields('vue_contactus_options'); ?>

                <!-- API Keys -->
                <div class="postbox">
                    <h3>Vue Contact US</h3>
                    <div class="inside">
                    <label for="vue_contactus_from_email">From Email Address:</label><br>
                        <input style="width: 50%;"
                                type="text" id="vue_contactus_from_email"
                                name="vue_contactus_from_email"
                                value="<?php echo get_option('vue_contactus_from_email'); ?>" /><br>

                        <label for="vue_contactus_from_name">From Name:</label><br>
                        <input style="width: 50%;"
                                type="text" id="vue_contactus_from_name"
                                name="vue_contactus_from_name"
                                value="<?php echo get_option('vue_contactus_from_name'); ?>" /><br>


                        <label for="vue_contactus_sendgrid_key">Sendgrid Key:</label><br>
                        <input style="width: 100%;"
                                type="text" id="vue_contactus_sendgrid_key"
                                name="vue_contactus_sendgrid_key"
                                value="<?php echo get_option('vue_contactus_sendgrid_key'); ?>" /><br>

                    </div>
                </div>
                <!-- end API Keys -->

                <?php submit_button(); ?>
            </form>
        </div>
    </div>

<?php } //end options page ?>


```

### Step 5: Create the page where the Contact Us form takes in

Let;s bring our dependencies.
We'll going to import the sendgrid-php library.

We do need a WordPress function. Since this file is called directly, outside of WordPress, we'll need to bring in WordPress real quick.


```php

// we need to pull in sendgrid-php api
require_once('sendgrid-php/vendor/autoload.php');

// SO WE NEED TO SUMMON WORDPRESS -- https://stackoverflow.com/a/19327058/4096078
define('WP_USE_THEMES', false); // Don't load theme support functionality
require_once('../../../wp-load.php');
```

Now that the dependencies are taken care of, let's get the form submission!
We'll check to see if we get a POST request.
We take the JSON, and convert it into a PHP Array.
We'll also get our WP Settings Page data, using the WP helper function `get_option`
ANd we'll pass all of that into our `send_contactus_email_sendgrid()`

```php 
if ($_SERVER['REQUEST_METHOD'] === 'POST' && empty($_POST)) {

   // get the header data, and decode it to PHP array
   $data = json_decode(file_get_contents("php://input"), true);

   // get wordpress options values
   $config["password"] = get_option('vue_contactus_sendgrid_key');
   $config["fromEmail"] = get_option('vue_contactus_from_email');
   $config["fromName"] = get_option('vue_contactus_from_name');

   send_contactus_email_sendgrid($data, $config);
}
```


Now here is where the magic happens.
We sanitize the inputs, otherwise we can have a little [Bobby Tables issue](https://xkcd.com/327/).


```php 
// COMES FROM HERE
// https://github.com/sendgrid/sendgrid-php
function send_contactus_email_sendgrid($data, $config) {

   try {

   // sanitize email content
   $filteredData = filter_var_array ($data, [
      'first name' => FILTER_SANITIZE_STRING,
      'last name' => FILTER_SANITIZE_STRING,
      'email' =>FILTER_VALIDATE_EMAIL,
      'company' => FILTER_SANITIZE_STRING,
      'department' => FILTER_SANITIZE_STRING,
      'subject line' => FILTER_SANITIZE_STRING,
      'messaage' => FILTER_SANITIZE_STRING
   ]);

   // compile email content
   $message = '';
   foreach ($filteredData as $input => $value) {
      if (is_array($value)) {
         $message .= implode(", ", $value);
      } else {
         $message .= "$input : $value <br />";
      }
   }

   $email = new \SendGrid\Mail\Mail();
   $email->setFrom($config["fromEmail"], $config["fromName"]);
   $email->setReplyTo($config["fromEmail"], $config["fromName"]);
   $email->setSubject($data['subject line']);
   $email->addTo($data['email'], $data['first name'] . ' ' . $data['last name']);
   $email->addContent("text/html", $message);

   $sendgrid = new \SendGrid($config["password"]);
   // $sendgrid = new \SendGrid('ERROR CHECKING');

   $response = $sendgrid->send($email);

   // sends to Vue App.
   // If it's 202, it's successful.
   print $response->statusCode();

} catch (Exception $e) {
      echo 'Caught exception: ' . $e->getMessage() . "\n";
   }

}
```

## Testing

Alright, let's test the Vue Contact us form.

As of right now, the `email` field is set to send it back to the user. 
I used a [TempMail](https://temp-mail.org/en/) service for testing.

I submitted, and it looks like this:
![](https://i.imgur.com/b6ci0tu.png)

## Conclusion

Big success.

### UNUSED AND DELETE

The primary form:
```vue

```



Text FIeld Compeonent

```vue
// inside app/src/components/input-text.vue
```

TextArea Input Compeonent
```vue
// inside app/src/components/input-textarea.vue
```

Checkbox Input Component
```vue
// inside app/src/components/input-checkbox.vue
```

Radio Input Component
```vue
// inside app/src/components/input-radio.vue
```
