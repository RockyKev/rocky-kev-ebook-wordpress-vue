# TITLE

## Introduction

## What we are doing:

In this project, we'll be creating a Vue Contact Us page that sends a submission to SendGrid API.

On submit, your data will be sent to a PHP backend file. 
When your data arrives in PHP, it then gets sent to a API which generates a success/fail. 
The API will automatically send a email to your choice.


### We could skip PHP, but we're not
We could easily skip the PHP backend and have Vue send the Contact Us directly to the API.

But the intention is to get data back into PHP and WordPress.

For example, maybe you: 
* Want to pass the data to the database
* OR want to use a PHP email library like PHPMailer().

Let's jump right into it.
## Process:

The focus is on PHP code. 

I've already generated a Contact Us app in Vue to save you time. 
We'll walk through the Contact Us in Step 3. And of course, you can modify the form values as you please.

## STEP BY STEP:

Step 1: Download my repo with a pre-built Vue Contact Us form

Step 2: Get your SendGrid API credentials

Step 3: Quickly understanding the Vue Contact Us form

Step 4: Generate our Contact Us Vue App as a shortcode

Step 5: Create a Contact Us WP Settings Page

Step 6: Create the Contact Us Backend Page

### Step 1: Download my repo with a pre-built Vue Contact Us form

Download my tutorial branch of the project. 
The Vue code is already set up. 
Within this tutorial, we'll be filling out the PHP files.

[Get the Repo here:](https://github.com/RockyKev/wordpress-vue-contact-us/tree/tutorial)

Inside the repo, it looks like:
```
.
└── wordpress-vue-contact-us/
    ├── app/          
    ├── vue-contact-us.php
    ├── vue-contact-us-submit.php
    └── vue-contact-us-settings-admin.php
```

Let's break those down:

`app/` folder
The Vue App that's pre-built with a Contact Us.
We'll cover the details in Step 3. 


```
 "scripts": {
    "postinstall": "npm install --prefix ./app",
    "build:app": "npm run build --prefix ./app",
    "serve:app": "npm run serve --prefix ./app"
  },
```

From within the root folder, you can `npm run postinstall`, which is a custom script command that will enter the `app` folder, and `npm install` the vue dependencies.

To build the project, you can `npm run build:app`. 
To run the code in dev mode, use `npm run serve:app`

Finally, the PHP Files: 

`vue-contact-us.php`
A blank PHP file for our WP plugin. When your WP plugin gets activated, this is the entrypoint.
This will also include code for the the contact us settings page, and code to pass PHP data that will pass to our Vue App.

`vue-contact-us-settings-admin.php`
This file contains code to generate the Plugin Settings page.

`vue-contact-us-submit.php`
This file takes the Vue Contact Us submission data. It then sanitizes the input, pass it to the SendGrid API, and fires off the email. If the submission fails, it'll send a error request back to Vue.

### Step 2: Get your SendGrid API credentials
#### Getting a SendGrid Account 

You must have a SendGrid account to get a API.
Sendgrid has a free version for developers.
You can [create a free Sendgrid account here.](https://sendgrid.com/free/)

After creating your account, you'll then create a Web API.
![](https://i.imgur.com/FF7yiHh.png)

These are my settings for this project.
![](https://i.imgur.com/yqx9ko0.png)

Save your API key somewhere safe. 
We'll be creating a Options page for our plugin, which will save the data in the database.

#### Using the SendGrid PHP Libbrary 

Next, inside your plugin, create a  `sendgrid-php/` folder.

Then download the [SendGrid PHP Library](https://github.com/sendgrid/sendgrid-php) and extract it into that folder. 

I also downloaded the latest version of this writing of the SendGrid PHP API Library (7.9.2).

Your repo should look like this: 
```
.
└── wordpress-vue-contact-us/
    ├── app/          
    ├── sendgrid-php/
    │   ├── examples/
    │   ├── lib/
    │   ├── static/
    │   ├── vendor/
    │   ├── ...
    │   ├── sendgrid-php.php
    │   └── ...
    ├── vue-contact-us.php
    ├── vue-contact-us-submit.php
    └── vue-contact-us-settings-admin.php
```

By default, Wordpress by default uses [wp_mail()](https://developer.wordpress.org/reference/functions/wp_mail/), which is a wrapper around [phpmailer()](https://github.com/PHPMailer/PHPMailer). 

There's are many of conveniences that the sendgrid-php packages has, like excellent documentation and providing error codes, things that require a lot more work if we use the wp_mailer() code.

### Step 3: Quickly understanding the Vue Contact Us form

Let's dig into the Vue Contact Us Form.

Here's how the form looks:
[screenshot]

Within the files:
```
└── wordpress-vue-contact-us/
    └── app/          
        ├── src/
        │   ├── assets/
        │   ├── components/
        │   │   ├── InputCheckbox.vue
        │   │   ├── InputRadio.vue
        │   │   ├── InputText.vue
        │   │   └── InputTextArea.vue
        │   ├── views/
        │   │   └── ContactUs.vue
        │   ├── App.vue
        │   └── main.js
        ├── package.json
        └── vue.config.js
```

Inside the `app/src/components` folder are for reusable component inputs.

* Text field
* TextArea field
* Radio Button
* Checkbox

Inside `app/src/views` folder is a single component called `ContactUs.vue` that controls the entire Contact Us form.


I also included a debug section, where you can look at the data before it's being sent. 

``` html
 <div class="debug">
      <h2>This userData gets sent to PHP</h2>
      <pre>
        <code>{{ userData }}</code>
        <code>Endpoint:{{endpoint}}</code>
        </pre>
    </div>
```

There's also some client-side validation built in.

When you click the submit button, the form will check if:

1. Your email is actually an email. 
2. If the required fields are filled out and not blank.
3. The checkboxes that are required are also checked.

If your submission meets all of those criterias, the data flows into the vue method `submitToPHP()`

That data sends a post request via AXIOS to submit to the `vue-contact-us-submit.php` file, which we'll create in Step 6.

That's enough about the Vue forms. 
Let's jump into the PHP side of things.
### Step 4: Generate our Contact Us Vue App as a shortcode

Inside your `vue-contact-us.php` file, let's generate the Vue App itself.

Next, we'll create a function to call our Vue App.

```php
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
```php
	// pass plugin directory
	$wp_plugin_uri = plugin_dir_url( __FILE__ );

	wp_localize_script('my-vue-app', 'wpPluginUri', $wp_plugin_uri);
```

Using this, we're passing the url directory so within our Contact Us Vue App, it'll take the `wpPluginUri` data, and use it as the endpoint that the form gets submitted to.


Also, let's include the a WP Settings Page, which we'll create in the next step.
```php
include_once('vue-contact-us-settings-admin.php');

function vue_contact_us() {
    // get Vue libs
    ...
}
```

Finally, let's put it on our WordPress site.

We can do that by using the shortcode: 
`[contact-us-vue]`


### Step 5: Create a Contact Us WP Settings Page

In order to use SendGrid API, we also need to send them our secret key.

DO NOT PUT YOUR SECRET KEY IN YOUR CODE.
> The researchers searched approximately 150 million entities across GitHub, GitLab, and Pastebin during a 30-day period in August and September to find the roughly 800,000 keys. Via [Dark Reading](https://www.darkreading.com/vulnerabilities---threats/research-finds-nearly-800000-access-keys-exposed-online/d/d-id/1338918)

We can store our secret key in the WordPress Database, and then call to it when we need it.

We can do that by creating a WP Settings page.

It'll look like this:

![](https://i.imgur.com/IKP0KI5.png)

(I prefer this method of generating menus, which I borrowed from [Building a Better WordPress Theme with the Settings API](https://ieg.wnet.org/2015/08/build-a-better-wordpress-theme-with-the-settings-api/#:~:text=add_option%20is%20used%20to%20declare,the%20values%20of%20the%20fields.) rather than the class-based approach, which I did in [BLOG POST 2](BLOG POST 2 DIRECT LINK TO THAT STEP)

There's three steps to making a Settings Page. 
1. Add the menu to the sidebar. 
2. Register the inputs so WordPress knows where to save it in the database.
3. Create a form inside the settings page to send the data to the database.


First, let's add the menu item to the WordPress sidebar.



```php
// inside `/vue-contact-us-settings-admin.php`
// This adds it to the WordPress Sidebar
function vue_contactus_add_theme_menu_item() {
    add_menu_page("Vue Contact Us Settings", "Vue Contact Us", "manage_options", "vue-contactus-settings", "vue_contactus_options_page", null, 99);
}

add_action("admin_menu", "vue_contactus_add_theme_menu_item");
```

Next, lets register the inputs so WordPress knows where to save it in the database.

We need 3 options.

A input for a `from email`
A input for a `from name`
And finally, a input to stash your sendgrid api key. 

```php 
// inside `/vue-contact-us-settings-admin.php`

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



Finally, lets create a form inside the settings page to send the data to the database.

```php 
// inside `/vue-contact-us-settings-admin.php`

// Build the theme options page
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

Once your page is set, visit it in the WordPress menu, and enter your Secret key!

Another win for security!



### Step 6: Create the Contact Us Backend Page

Let's go down our checklist.
We have a Vue App that generates the Contact us form.
We created a PHP Plugin that outputs the Vue App 
We added the code to a WordPress page. 
We also created a Settings Page where we submitted our SendGrid Secret Key.

We now need to take the data generated from the Contact Us form and do something with it.

We'll do that in this file.

`/vue-contact-us-submit.php`


First, let's bring our dependencies.
We'll going to import the sendgrid-php library.

```php
// inside `/vue-contact-us-submit.php`

// we need to pull in sendgrid-php library
require_once('sendgrid-php/vendor/autoload.php');
```

Remember that this file is called directly by a axios call in Step 3. 

Since this file is called directly, it lives outside of WordPress. 

we'll need to bring in WordPress real quick to use some nice WordPress features.

```php 
// inside `/vue-contact-us-submit.php`

// SO WE NEED TO SUMMON WORDPRESS -- https://stackoverflow.com/a/19327058/4096078
define('WP_USE_THEMES', false); // Don't load theme support functionality
require_once('../../../wp-load.php');
```

Let's get the form submission!

As noted before, we are calling this file directly. 

Here's the process: 

1. We'll check to see if we get a POST request.

2. We take the JSON, and convert it into a PHP Array.

3. We'll also get our WP Settings Page data, using the WP helper function `get_option`


```php 
// inside `/vue-contact-us-submit.php`

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

Finally, we'll pass all of that into our `send_contactus_email_sendgrid()`


Here is where the magic happens.
We sanitize the inputs, otherwise we can have a little [Bobby Tables issue](https://xkcd.com/327/).

```php 
// inside `/vue-contact-us-submit.php`

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

