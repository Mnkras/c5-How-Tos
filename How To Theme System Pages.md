How To Theme System Pages
=========
By: Michael Krasnow
---------

concrete5 allows theme developers to theme almost every aspect of the front end very easily.

A standard theme will have several files:

- `description.txt` - The name and description of the theme
- `default.php` - Default pagetype template
- `view.php` - Default single page template

We are really interested in the `view.php`, that is used to theme single pages such as the checkout page in eCommerce package. Some core system pages such as the Page Not Found page can be a little more tricky.

In the core there is a file called `theme_paths.php` (located in `/concrete/config`) which sets the core theme on the following pages:

- /page\_forbidden
- /page\_not\_found
- /login
- /register
- /maintenance_mode

This prevents the theme from taking over by default.

If it is a non-packaged theme there is only one way to theme those pages. In the `/config` directory of your site there will be a file called `site_theme_paths.php`, this file allows you to override anything set in the `theme_paths.php` file mentioned earlier.

In this file there will be some commented out code:

	$v = View::getInstance();

	$v->setThemeByPath('/login', "yourtheme");
	$v->setThemeByPath('/page_forbidden', "yourtheme");
	$v->setThemeByPath('/register', "yourtheme");
	
In this code we are manually setting the theme for a page. You can simply un-comment the code and replace `yourtheme` with your theme's handle and it will instantly take effect on your site.