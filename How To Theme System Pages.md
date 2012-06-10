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

If your theme is in a package you can solve this a little bit more elegantly. (You can still use the above method even if the theme is in a package.)

Below is an example package controller:

	<?php defined('C5_EXECUTE') or die("Access Denied.");
	
	class MyThemePackage extends Package {
	
		protected $pkgHandle = 'my_theme';
		protected $appVersionRequired = '5.5.2';
		protected $pkgVersion = '1.0';
	
		public function getPackageName() {
			return t("My Theme");
		}
	
		public function getPackageDescription() {
			return t("A wonderful theme this is!");
		}
		
		public function install() {
			$pkg = parent::install();
			PageTheme::add('my_theme', $pkg);
		}
		
	}

Right now all this does is install the theme.

What we are going to do is add an event to set the theme on select pages.

First we have to a new function to our package controller.

	public function on_start() {
	
	}
	
This function is fired when concrete5 starts running.

In this function we are going to extend the `on_start` event in concrete5.

	public function on_start() {
		Events::extend('on_start', get_class($this), 'setTheme', __FILE__);
	}
	
Now we are going to add the `setTheme` function to our controller.

	public function setTheme() {
		Loader::model('page_theme');
		$theme = PageTheme::getByHandle('my_theme');
		$site = $theme->getSiteTheme();
		if($theme == $site) { //we check to make sure this theme is applied to the site.
			$pages = array('/login', '/page_not_found', '/register'); //these are the pages we wan't to theme
			$view = View::getInstance();
			foreach($pages as $page) {
				$view->setThemeByPath($page, $theme->getThemeHandle());
			}
		}
	}
	
Replace the pages array with an array of pages like those above, and `my_theme` with your theme handle.

So our controller will now look like this:

	<?php defined('C5_EXECUTE') or die("Access Denied.");
	
	class MyThemePackage extends Package {
	
		protected $pkgHandle = 'my_theme';
		protected $appVersionRequired = '5.5.2';
		protected $pkgVersion = '1.0';
	
		public function getPackageName() {
			return t("My Theme");
		}
	
		public function getPackageDescription() {
			return t("A wonderful theme this is!");
		}
		
		public function install() {
			$pkg = parent::install();
			PageTheme::add('my_theme', $pkg);
		}
		
		public function on_start() {
			Events::extend('on_start', __CLASS__, 'setTheme', __FILE__);
		}
		
		public function setTheme() {
			Loader::model('page_theme');
			$theme = PageTheme::getByHandle('my_theme');
			$site = $theme->getSiteTheme();
			if($theme == $site) { //we check to make sure this theme is applied to the site.
				$pages = array('/login', '/page_not_found', '/register'); //these are the pages we wan't to theme
				$view = View::getInstance();
				foreach($pages as $page) {
					$view->setThemeByPath($page, $theme->getThemeHandle());
				}
			}
		}
	
	}
	
And thats it!