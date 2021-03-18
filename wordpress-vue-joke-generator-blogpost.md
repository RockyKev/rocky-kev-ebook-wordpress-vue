# TITLE

## Introduction



## What we are doing
Passing data to your Vue Plugin

WordPress Settings Page. 


Changing settings changes that too. 

[screenshot of it]



## Process

1. Set up a Vue-powered WordPress plugin
2. Fetch content from the jokes API.
3. Prepare your Vue App to take PHP data
4. Pass a placeholder PHP data object to the Vue Plugin
5. Create a settings page

## Step-by-step

### Step 1: Set up a Vue-powered WordPress plugin

OPTION 1: Follow my previous blog post. --- LINK


OPTION 2: Download and set this up yourself.

[LINK TO PROJECT ON GITHUB]

How your site should look like. 
```
.
└── wp-content/
    └── plugins/
        └── vue-joke-generator/                 // This is the Plugin you will create
            ├── app/                            // This is the Vue Side
            │   ├── dist/
            │   ├── node_modules/
            │   ├── public/
            │   ├── src/
            │   │   ├── assets/
            │   │   ├── components/
            │   │   ├── App.vue
            │   │   └── main.js
            │   ├── ...
            │   ├── package.json
            │   └── vue.config.php
            └── vue-app-joke-generator.php        // This is the WordPress PHP Side
```

Some things to recognize:

First, `vue-app-joke-generator.php` is your PHP entry point. 

This file will: 
* Load your plugin into WordPress
* Register all the scripts needed for your Vue App to work
* Create a div element that your Vue App will hook into
* Call the WordPress Settings API to generate a Plugin Settings Page
* Pass the WordPress Settings data to your Vue App
 
Then, in `app/` are your Vue Files. 

Within your Vue Project, you will: 
* Generate the Frontend view
* Get the `php_data` from your PHP Entry point
* Grab the `php_data` and mount it as a prop to your Vue App
* Fetch the Joke API with custom paramters based on your `php_data`
* Return a joke 





### Step 2: Fetch content from the jokes API.


We'll be building a separate component called `JokeApp.vue`. 
It'll make a fetch to the Joke API, and return a joke.


#### Inside App.vue

We'll be modifying it later to pass PHP data via props. 
But for now, let's keep it simple.

```vue
// vue-joke-generator/app/src/App.vue
<template>
  <div>
      <JokeApp  />
  </div>
</template>

<script>
import JokeApp from "./components/JokeApp.vue";

export default {
  name: "App",
  components: {
    JokeApp
  }, 
};
</script>
```

#### Inside our new component JokeApp.vue

In the `<template></template>`, we'll lay out our frontend content.
We're doing some [vue conditional rendering](https://vuejs.org/v2/guide/conditional.html) based on the state of the API Call.

States: 
error - If there's an error with the fetch
loading - While the fetch is happening
else - Show the results of the fetch.

The joke API gives us metadata as well. I added a `v-show` and a `v-for` loop, just for practice. It's not necessary. But if you never seen them before, they're fantastic tools in Vue.

```vue
// vue-joke-generator/app/src/components/JokeApp.vue
<template>
  <div>
    <div v-if="errored">Sorry. API call failed for some reason.</div>

    <div v-else>
      <div v-if="loading">Loading...</div>

      <div v-else>
        <h2>It's YUK YUK TIME.</h2>

        <p v-html="content"></p>

        <!-- Random extra content because why not -->
        <div v-show="!response.error">
          <hr />
          METADATA from api:
          <ul>
            <li>ID: {{ response.id }}</li>
            <li>language: {{ response.lang}}</li>
            <li>type: {{ response.type}}</li>
            <li>category: {{ response.category}}</li>            
          </ul>
          Flags: 
          <ul>
            <li v-for="(value, name) in response.flags" :key="name">{{ name }} : {{ value }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>
```


In the `<scripts></scripts>` section, we'll make our fetch command.
```vue
// vue-joke-generator/app/src/components/JokeApp.vue

<script>
export default {
  name: "JokeApp",
  props: {
    params: Object,
  },
  data: function () {
    return {
      response: null,
      loading: true,
      errored: false,
      content: "",
    };
  },
  mounted() {
    this.fetchAPI();
  },
    methods: {
        fetchAPI() { ... },
        addFetchParams() { ... },
        showResults() { ... },
  },
};
</script>
```

I added 3 placeholder methods, which we'll fill below.

But let's quickly talk about it.

`fetchAPI()` is the method that makes the API call to the Joke API.
`addFetchParams()` is the method that will take props data (which will come from your PHP file, the WordPress Settings data), and append it to the fetch URL.
`showResults()` controls what the user sees, based on the response of the fetch.

Let's start with the fetchAPI() method:

Nothing should be too surprising.

```js
  methods: {
    fetchAPI() {
      const url =
        "https://v2.jokeapi.dev/joke/" + this.addFetchParams();

      // console.log("url sent", url);

      fetch(url)
        .then((response) => response.json())
        .then((response) => {
          this.response = response;
        })
        .catch((error) => {
          console.error(error);
          this.error = true;
        })
        .finally(() => {
          console.log(this.response);
          this.showResults();

          this.loading = false;
        });
    },
    addFetchParams() { ... },
    showResults() { ... },
  },
 ```


Next, we'll be adding our Params code. 

Within the JokeAPI, you can include parameters to better filter your call.

Looking at this URI:
```
https://v2.jokeapi.dev/joke/Programming?blacklistFlags=nsfw,explicit&type=single

Categories: 

```
You can (play with the generator itself!)[https://sv443.net/jokeapi/v2/#try-it]

![](https://i.imgur.com/jr1DiBD.png)


 ```js
  methods: {
    fetchAPI() { ...  },
    addFetchParams() {
        // get category settings and add to param
        const categories = Object.keys(this.params.category)
                                .filter((key) => this.params.category[key]);
        const categoriesOutput = categories?.length ? categories.toString() + "?" : "Any?";

        // get blacklist settings and add to param
        const blacklist = Object.keys(this.params.blacklist)
                                .filter((key) => this.params.blacklist[key]);
        const blacklistOutput = blacklist?.length ? "blacklistFlags=" + blacklist.toString() + "&" : "";

        // get jokeType settings and add to param
        const jokeType = this.params.jokeType;
        const jokeTypeOutput = jokeType !== "any" ? "type=" + jokeType + "&" : "";

        // get range -- if any of them are blank, ignore
        const rangeFrom = this.params.range.from;
        const rangeTo = this.params.range.to;
        const rangeOutput =
            (rangeFrom !== null) && (rangeFrom !== undefined) && rangeTo
            ? "idRange=" + rangeFrom + "-" + rangeTo : "";

            return categoriesOutput + blacklistOutput + jokeTypeOutput + rangeOutput;
    },
    showResults() { ... },
  },
```

Once the fetch call is successful, we need to do something with that data. 

If you make a direct call to the [JokeAPI](https://v2.jokeapi.dev/joke/any?), you get this json response:
```
{
    "error": false,
    "category": "Programming",
    "type": "single",
    "joke": "Debugging is like being the detective in a crime movie where you're also the murderer at the same time.",
    "flags": {
        "nsfw": false,
        "religious": false,
        "political": false,
        "racist": false,
        "sexist": false,
        "explicit": false
    },
    "id": 42,
    "safe": true,
    "lang": "en"
}
```

There's two things to notice above. There's a `error` element, and a `type` element.

The fetch call could be successful, but the JokeAPI could return a `error` like "No matching joke found". 
And the `type` element can be either "single" or "twopart" jokes, which have different ways to display the content.

That's what the `showResults()` method will solve.

```js
  methods: {
    fetchAPI() { ...  },
    addFetchParams() { ... },
    showResults() {

      // if it's an error, show it. Else show the joke type
      if (this.response.error) {
        this.content = this.response.message;
      } else if (this.response.type === "twopart") {
        this.content = `
          SETUP: ${this.response.setup} <br /><br /><br />
          DELIVERY: ${ this.response.delivery }
        `
      } else if (this.response.type === "single") {
        this.content = this.response.joke;
      } else {
        this.content = "Something happened and I'm not quite sure.";
      }

    }
  },
```

<!-- TODO: Test to see if this works? -->

It won't work quite just yet. We still need to set up a bit of the PHP and WordPress side of things.

### Step 3. Prepare your Vue App to take PHP data

build it here.


In your main PHP plugin file - we need to generate PHP data for the plugin to grab.
We'll keep it simple for now by hardcoding the object.
In the next step, we'll have it generated from the WordPress Settings API.

Update your vue_fetch_jokes() function to the code below.

```php 
// Inside vue-app-joke-generator.php
function vue_fetch_jokes() {

	$phpData = [
		'category' => [
			'Programming' => "",
			'Miscellaneous' => "on",
			'Pun' => "",
			'Spooky' => "",
			'Christmas' => ""
		],
		'blacklist' => [
			'nsfw' => "on",
			'religious' => "",
			'political' => "on",
			'racist' => "",
			'sexist' => "",
			'explicit' => ""
		],
		'joketype' => 'any',
		'joke_range1' 0,
		'joke_range2' 20,
	];

	// get Vue libs 
	wp_register_script('vue-app-vendors',  plugins_url('app/dist/js/chunk-vendors.js', __FILE__), array(), '1.0.0');
	wp_register_script('my-vue-app', plugins_url('app/dist/js/app.js', __FILE__), array('vue-app-vendors'), '1.0.0');

	// pass php data
	// wp_localize_script('my-vue-app', 'phpData', $phpData);
	wp_localize_script('my-vue-app', 'phpData', $php_data);	

	// pass the scripts to WP
	wp_enqueue_script('vue-app-vendors');
	wp_enqueue_script('my-vue-app');
	wp_enqueue_style('my-vue-app',  plugins_url('app/dist/css/app.css', __FILE__),   array(),  '1.0.0');

	return '<div id="app"></div>';
}
```

Notice that the `$phpData` object is essentially using TRUE/FALSE values, but as `"on"` and `""`.
The way WordPress stores WordPress Settings API data is using that. We're shaping our data to match.

`	wp_localize_script('my-vue-app', 'phpData', $php_data);`
This lone of code is sending our PHP object out to the `window` object in the browser. [MDN Web Docs on Window](https://developer.mozilla.org/en-US/docs/Web/API/Window) That will allow our Vue App to see our PHP data and grab it.

We are also calling that data `phpData`. You can name it ANYTHING. But the name you use is what Vue is expecting to get.

### Step 4. Pass a placeholder PHP data object to the Vue Plugin

Now let's modify our Vue App to receive the data. 
```js
// Inside app/src/main.js

import { createApp } from "vue";
import App from "./App.vue";

// createApp(App).mount("#app");
createApp(App, { ...window }).mount("#app");

```
There's that `window` object again! This time, we're passing it into Vue.
Let's check to see if that data exists!

```vue
//  Inside app/src/App.vue

<template>
  <div>
    <JokeApp :params="phpData" />
  </div>
</template>

<script>
import JokeApp from "./components/JokeApp.vue";

export default {
  name: "App",
  components: {
    JokeApp
  }, 
  props: {
    phpData: Object,
  },
  mounted() {
    console.log(this.phpData);
  },
};
</script>
```
We set up a `mounted()` lifescycle method to output the data that Vue received. 
We called the data `phpData`, and if it's done correctly -- 
you should now see it outputted in console.log, as well as your settings changing the Fetch call!

[SCREENSHOT]

This is great! Now we have PHP data passing over to our Vue App!

Our next step is to pass USER-MODIFIED data to our Vue App. 

### Step 5. Create a settings page

We're now going into WordPress territory. 

First, let's create a admin-settings page at the root directory.

How your site should look like. 
```
.
└── wp-content/
    └── plugins/
        └── vue-joke-generator/                 // This is the Plugin you will create
            ├── app/                            // This is the Vue Side
            └── vue-app-joke-generator.php        // This is the WordPress PHP Side
            └── vue-app-admin-settings.php        // This is the new file we made         
```


Second, this is the class we're adding into the project.

NOTE: This was borrowed heavily from [Twilio's blog post on building a custom WordPress Plugin](https://www.twilio.com/blog/build-custom-wordpress-plugin-send-sms-twilio-programmable-messaging). 

```php
// Inside vue-app-admin-settings.php

class SettingsForVuePlugin {

    // 1 - identify the name of the plugin
    private $pluginName = "vue-jokes";
    private $pluginSettingsName = "vue-jokes-settings-page";

    // 2 - Create the Form's UI
    // and name the page
    public function displaySettingsPage() {
        ?>
            <form method="POST" action="options.php">
                <?php
                settings_fields($this->pluginName);
                do_settings_sections($this->pluginSettingsName);
                submit_button();
                ?>
            </form>
        <?php
    }

    // 3 - Add it to WordPress, so it can be seen as a option
    public function addVueJokesAdminOptions() {
        $pageTitle = "Vue Jokes | Settings Page";
        $menuTitle = "Vue Jokes";

        // https://developer.wordpress.org/reference/functions/add_options_page/
        // add_options_page( string $page_title, string $menu_title, string $capability, string $menu_slug, callable $function = '', int $position = null )
        add_options_page(
            $pageTitle,
            $menuTitle,
            "manage_options",
            $this->pluginName,
            [$this, "displaySettingsPage"]
        );
    }

    // 4 - create the section Text
    public function sectionTextForJokeOptions() {
        echo '<p>Check the types of jokes you want.</p>';
    }

    public function sectionTextForAPI() {
        echo '<p>Change the API settings here.</p>';
    }


    // 5 - Register the fields
    // Also so when we hit save, it saves it in the WP Database
    public function saveVueJokesAdminOptions() {
        register_setting(
            $this->pluginName,
            $this->pluginName,
            [$this, "validatePluginOptions"]
        );

        // two sections

        // TODO: Maybe Separate them as separate sections/items?

        // adds a section with a description. 
        // add_settings_section( string $id, string $title, callable $callback, string $page )
        add_settings_section(
            $this->pluginName . "-joke-options",
            "Joke Category Settings",
            [$this, "sectionTextForJokeOptions"],
            $this->pluginSettingsName,
        );

        // adds the field itself
        // add_settings_field( string $id, string $title, callable $callback, string $page,
        //                     string $section = 'default', array $args = array() )
        // category --> checkboxes 
        add_settings_field(
            "joke_category",
            "Joke Category (Show Only. If none are checked, then all are selected)",
            [$this, "inputJokeOptionsCategory"], // has to be a callback
            $this->pluginSettingsName,
            $this->pluginName . "-joke-options"
        );

        // blacklist --> checkboxes 
        add_settings_field(
            "joke_blacklist",
            "Joke Blacklist (Remove These types of jokes)",
            [$this, "inputJokeOptionsBlacklist"],
            $this->pluginSettingsName,
            $this->pluginName . "-joke-options"
        );

        add_settings_section(
            $this->pluginName . "-api",
            "Joke API Settings",
            [$this, "sectionTextForAPI"],
            $this->pluginSettingsName,
        );

        // joketype --> radio (one, other, or both)
        add_settings_field(
            "joke_type",
            "Joke Type",
            [$this, "inputAPIJokeType"],
            $this->pluginSettingsName,
            $this->pluginName . "-api",
        );

        // jokerange -> input 1
        add_settings_field(
            "joke-range1",
            "Jokes - From:",
            [$this, "inputAPIJokeRange1"],
            $this->pluginSettingsName,
            $this->pluginName . "-api"
        );

        // jokerange -> input 2
        add_settings_field(
            "joke-range2",
            "Jokes - To:",
            [$this, "inputAPIJokeRange2"],
            $this->pluginSettingsName,
            $this->pluginName . "-api"
        );
    }

    // 6 - Create the inputs
    public function inputJokeOptionsCategory() {

        $categoryList = [
            "programming",
            "miscellaneous",
            "pun",
            "spooky",
            "christmas"
        ];

        $options = get_option($this->pluginName);

        $html = '';

        foreach ($categoryList as $item) {
            $id = $this->pluginName . "[category-$item]";

            $checked = ($options["category-$item"]) ? ' checked="checked"  ' : '';

            $html .= "<label><input $checked id='$id' name='$id' type='checkbox' />$item</label><br />";
        };
        echo $html;
    }

    public function inputJokeOptionsBlacklist() {
        $options = get_option($this->pluginName);

        $blackList = [
            "nsfw",
            "religious",
            "political",
            "racist",
            "sexist"
        ];

        $html = '';

        foreach ($blackList as $item) {
            $id = $this->pluginName . "[blacklist-$item]";

            $checked = ($options["blacklist-$item"]) ? ' checked="checked"  ' : '';

            $html .= "<label><input $checked id='$id' name='$id' type='checkbox' />$item</label><br />";
        };

        echo $html;
    }


    public function inputAPIJokeType() {
        $options = get_option($this->pluginName);
        $inputName = $this->pluginName . '[joketype]';

        $choices = ["any", "single", "twopart"];
        $html = "";

        foreach ($choices as $choice) {
            $checked = ($options['joketype'] === $choice) ? ' checked="checked" ' : '';
            $html .= "<label><input " . $checked . " value='$choice' name='$inputName' type='radio' /> $choice</label><br />";
        }


        echo "<fieldset>$html</fieldset>";
    }

    public function inputAPIJokeRange1() {
        $options = get_option($this->pluginName);
        echo "
                <input 
                    id='$this->pluginName[joke-range1]'
                    name='$this->pluginName[joke-range1]'
                    size='20'
                    type='text'
                    value='{$options['joke-range1']}'
                    placeholder='0'
                />
            ";
    }

    public function inputAPIJokeRange2() {
        $options = get_option($this->pluginName);
        echo "
                <input 
                    id='$this->pluginName[joke-range2]'
                    name='$this->pluginName[joke-range2]'
                    size='20'
                    type='text'
                    value='{$options['joke-range2']}'
                    placeholder='100'
                />
            ";
    }


    // 7 - Validate/Sanitize text 
    public function validatePluginOptions($input) {

        $formElements = [
            "category-programming",
            "category-miscellaneous",
            "category-pun",
            "category-spooky",
            "category-christmas",
            "blacklist-nsfw",
            "blacklist-religious",
            "blacklist-political",
            "blacklist-racist",
            "blacklist-sexist",
            'joketype',
            'joke-range1',
            'joke-range2'
        ];

        foreach ($formElements as $element) {
            $newinput[$element] = trim($input[$element]);
        }

        return $newinput;
    }
}
```
There's a lot to explain that will triple the size of this blog post. For the sake of brevity, I added comments to explain as much as I could.

Again, reference the [Twilio tutorial](https://www.twilio.com/blog/build-custom-wordpress-plugin-send-sms-twilio-programmable-messaging) for more details about how certain things work.

I'm not a fan of the WordPress Settings API, and will write a future blogpost to refactoring the settings using Vue instead.

Next, you'll need to call your Settings Page.

```php 
// create a settings page
include 'vue-app-admin-settings-page.php';
$settingsInstance = new SettingsForVuePlugin();

add_action("admin_menu", [$settingsInstance, "addVueJokesAdminOptions"]); // hook into to WP
add_action("admin_init", [$settingsInstance, "saveVueJokesAdminOptions"]); // hook into saving values

```

If everything is working, this should now be able to see this.

[Screenshot of Settings Page]


Where does your data go after you save it?

By default, it'll appear in the options table in your WordPress data.
![](https://i.imgur.com/sKQLoby.png)

But - this data isn't being used in our plugin just yet. Let's fix that.

## Step 6: Passing the Settings data over

`$options_data = get_option("vue-jokes");`

When you use the [get_option()](https://developer.wordpress.org/reference/functions/get_option/) WordPress API, it returns your data. 


How it looks:
```json
  {
  "blacklist-nsfw": "",
  "blacklist-political": "",
  "blacklist-racist": "on",
  "blacklist-religious": "",
  "blacklist-sexist": "",
  "category-christmas": "on",
  "category-miscellaneous": "on",
  "category-programming": "on",
  "category-pun": "on",
  "category-spooky": "on",
  "joke-range_1": "1",
  "joke-range_2": "10",
  "joketype": "any"
  }
```
How we want it:
```json
{
  "category": {
    "Programming": "",
    "Miscellaneous": "on",
    "Pun": "",
    "Spooky": "",
    "Christmas": ""
  },
  "blacklist": {
    "nsfw": "on",
    "religious": "",
    "political": "on",
    "racist": "",
    "sexist": "",
    "explicit": ""
  },
  "joketype": "any",
  "joke_range1": 0,
  "joke_range2": 20
}

```

We're going to write helper functions to reshape the data to match our previous setup.

```php 
// inside vue-app-joke-generator.php

// Helper functions
function replace_key_names($array, $search_string, $replace_with) {
    $matches = [];

    foreach ($array as $key => $value) {
        if (strstr($key, $search_string)) {
			$newKey = str_replace($search_string, $replace_with, $key);
            $matches[$newKey] = $value;
        }
    }

    return $matches;	
}

function get_values_with_key($array, $search_string) {
    $matches = [];

    foreach ($array as $key => $value) {
        if (strstr($key, $search_string)) {
            $matches[$key] = $value;
        }
    }

    return $matches;
}

// Borrowed from https://wordpress.stackexchange.com/a/137889/132722
function check_if_option_exists($name, $site_wide = false){
    global $wpdb; 
    return $wpdb->query("SELECT * FROM ". ($site_wide ? $wpdb->base_prefix : $wpdb->prefix). "options WHERE option_name ='$name' LIMIT 1");
}

// Invoke Shortcode
function vue_fetch_jokes() { ... }
}
```

And below is the updated function.

```php 
// inside vue-app-joke-generator.php

function vue_fetch_jokes() { 

	// shape the array 
  if (check_if_option_exists("vue-jokes")) {

    $options_data = get_option("vue-jokes");

    // reshape the array into it's own pieces
    $blacklist_array = get_values_with_key($options_data, "blacklist-");
    $category_array = get_values_with_key($options_data, "category-");

    // clean up
    $blacklist_array = replace_key_names($blacklist_array, "blacklist-", "");
    $category_array = replace_key_names($category_array, "category-", "");

    $php_data = [
      'category' => $category_array,
      'blacklist' => $blacklist_array,
      'jokeType' => $options_data['joketype'],
      'range' => [
        'from' =>  $options_data['joke-range1'],
        'to' => $options_data['joke-range2']
      ],
    ];
      
    // pass php data
    // wp_localize_script('my-vue-app', 'phpData', $phpData);
    wp_localize_script('my-vue-app', 'phpData', $php_data);	
  }

	// get Vue libs 
	wp_register_script('vue-app-vendors',  plugins_url('app/dist/js/chunk-vendors.js', __FILE__), array(), '1.0.0');
	wp_register_script('my-vue-app', plugins_url('app/dist/js/app.js', __FILE__), array('vue-app-vendors'), '1.0.0');

	// pass the scripts to WP
	wp_enqueue_script('vue-app-vendors');
	wp_enqueue_script('my-vue-app');
	wp_enqueue_style('my-vue-app',  plugins_url('app/dist/css/app.css', __FILE__),   array(),  '1.0.0');

	return '<div id="app"></div>';
```

## Conclusion

Any changes from the settings page gets passed to WordPress's Database. 
Then before we create our Vue App, PHP will pass the settings data as a JS object in the `window`.
Our Vue App will see that data and load it on our App.
