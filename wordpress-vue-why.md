

# Why Vue and WordPress

WHY VUE

This is a very opinionated post. This is why I am doing this.

Key word is I. 

I also want to address that the purpose of code is to make things happen. It's a tool, like a hammer or a wrench. To defend a tool as one without faults is silly. Your tools won't love you back. Swearing allegience to your tool and putting it on the altar is not only childish, but foolish. Do you love your hammer, or do you love that you built a house with it?

As a developer, I use tools. And a developer, I have preferences. I prefer my code to look a specific way. I prefer to code like I would write a essay or a letter -- clearly and with simple words. I prefer empty lines and open spaces.

To create a theme in PHP is straightforward. It's clean to write. It's straightforward. 

But the tooling isn't there. I can't imagine creating a beautiful WordPress theme out of pure PHP and vanilla HTML/CSS/JS. At a basic level, I'd need SASS. At a higher level, how would I control my JS and minimize it? How would I create vendor-prefixes for different browsers? How do I bring in third-party libraries directly into my code without making another load request?

And the more I think about these basic comforts of modern development, the more I end up having to use a build tool like Webpack. 

For modern development, we now have two directions -- React or Vue. They both have their pros and cons. They both are very powerful frameworks. 

As a React developer, React solves a lot of problems. I can build beautiful reusable components. I can focus on code and design. But there's something I always disliked about React, something I also disliked about PHP. 

PHP as a templating language is fugly.

```php 
<?php if ($variableExists): ?>
    <span class="<?php echo $element->small; ?>">Subtitle here</span>
<?php endif; ?>
```

This isn't very ugly. But it can get there. 

React as JSX is pretty damn ugly. 

```js

	<MediaUpload
								title={ __( 'Select poster image' ) }
								onSelect={ onSelectPoster }
								allowedTypes={
									VIDEO_POSTER_ALLOWED_MEDIA_TYPES
								}
								render={ ( { open } ) => (
									<Button
										isPrimary
										onClick={ open }
										ref={ posterImageButton }
										aria-describedby={
											videoPosterDescription
										}
									>
										{ ! poster
											? __( 'Select' )
											: __( 'Replace' ) }
									</Button>
								) }
							/>
```

This is actual code from WordPress block library. https://github.com/WordPress/gutenberg/blob/trunk/packages/block-library/src/video/edit.js

Bootcamps all over the world teach HTML, and then say, "Yeah now learn REACT!" and watch as a large percentage of developers sink or swim and question if they are in the right industry. 


And since this is JSX, writing CSS is a pain too with requiring to use `className` and the JS equivelent of class properties. Or you can dig into Motion, CSS-in-JS, CSS Modules, or whatever workaround. Sorry bootcamp students that we had to teach you HTML, CSS, JS, and THEN JSX, a JS version of CSS, and the React ecosystem. 6 different languages. 


Now we get to Vue.

Vue said, "What if we took all the cool things about React, but let you use vanilla HTML, CSS, and JS?"

Vue shorten the curve. All the power of React, but using basic building blocks that we were all familiar with. 

So why Vue and WordPress?

Because coding should be enjoyable. 



WHY AM I DOING THIS:

In 2016, Matt Mulewig (PROOF NAME PLZ), the creator of WordPress, made a call to action for WordPress developers to "Learn Javascript deeply". 

For the next few years, WordPress is evolving out of it's PHP shell into modern development. Besides a little hiccup with React licenses in 2018, WordPress has now decided that React is the framework of choice. 

I like React a lot. But I also like Vue. 

And as a Laravel developer (which uses a Laravel backend and Vue frontend), I was disappointed to see WordPress go in the direction of React, as opposed to a agnostic direction. 

React make sense business-wise -- React solved a LOT of problems and ushered us into this new era of modern development. ANd versus Vue, there's a significantly larger population of React developers (by 7 out of 10 devs are using React), a larger set of tools, and more importantly, a larger body of experience and documentation thanks to React's main maintainer -- Facebook.

But what about the 3 out of 10 developers using Vue? 

WordPress is almost 40% of the internet (As of DATE + link). 

And frankly, I like Vue. 


-----
The future of WordPress and Vue

Blocks are the future of WordPress. 
The hope is that one day, a user can build a WordPress site PURELY with Blocks. 

Blocks are written with a tiny bit of PHP, but mostly in JSX. (React)

How does Vue look in a React-based Block Library? 

Well frankly, in a really really shitty position. 

I spent a two weeks spinning in circles, building React-based Blocks and trying to find a way to make it Vue-like. Unfortunately, I wasn't lucky. 

But not all is lost.

Gutenberg was released in 2018 (and with it, WordPress Blocks), and Full-site editing is coming closer (expected 2022). 

Blocks are at the toddler stage of life. 
Roughly a few months ago (Sept 2020), WordPress released Blocks API 2. I have found ZERO tutorials on how to use it. THe Blocks documentation was serverly lacking. ANd even a documentation writer for WordPress made a tutorial in March 2021 using the original API. 

In other words -- WordPress still has a long way to go for Block development.

## So why am I writing this book again?

Because this Version 1 -- it's going to be outdated. It's going to be rough. It's going to feel like I wrote a WordPress book in 2020 that's not the cutting edge WordPress.

Yeah.

But in time, maybe it won't be that way. 
And for now, this is the best Vue developers have. 

So let's get to work.

















