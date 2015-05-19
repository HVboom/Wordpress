# [WordPress](https://wordpress.org)
Guide to multisite WordPress installation on a [SwitchPlus](https://www.switchplus.ch) account

## Installation

### Domain setup
* I use a [subdomain](https://app.switchplus.ch/reg/pr03/product-manage/index.html?lid=de&page=config) to host my multisite WordPress installation pointing to a dedicated subdirectory, e.g. _Homepage/WordPress_

### DB setup
* Login to [SwitchPlus](https://app.switchplus.ch/reg/pr03/product-manage/index.html?lid=de&page=database) and create a new database
  * Current MySQL version is 5.6.19 (25.04.2015)
* Use the phpMyAdmin panel for your new database and set on the _Operation_ tab the collation option to `utf8mb4_general_ci` (new default for WordPress 4.2)

### WordPress setup
* Download latest [*.tar.gz](https://wordpress.org/download/) file and upload it to the _WordPress_ folder
* SSH into your account and extract the tar-file: `tar -xzvf wordpress-4.2.tar.gz`
* Rename the new directory _wordpress_ to _core_: `mv wordpress core`
* Hide the real configuration in a non-accessable configuration directory by defining a _wp-config.php_ file outside the _core_ directory: `echo 'include('config/wp-config.php');' > wp-config.php`
  * Copy the sample configuration file into the _config_ directory: `cp core/wp-config-sample.php config/wp-config.php`
  * Here you find a complete list of option for [wp-config.php](http://codex.wordpress.org/Editing_wp-config.php)
* Adjust the database settings within this file to point to your newly created MySQL DB
  * Set the _DB_CHARSET_ to `utf8mb4` and _DB_COLLATE_ to `utf8mb4_general_ci`
* Use the online [generator](https://api.wordpress.org/secret-key/1.1/salt/) to get a set of secret keys

####  Separate the content from the WordPress application (see [Codex](http://codex.wordpress.org/Giving_WordPress_Its_Own_Directory#Using_a_pre-existing_subdirectory_install))
* Additional entries added to the _wp-config.php_:
  ```PHP
/**
 * Setup site specific constants
 *
 *   Location of the sites home
 *   Location of the WordPress application files
 *   Move content folder to it's own
 */
define( 'WP_HOME', 'http://wp.hvboom.ch' );
define( 'WP_SITEURL', WP_HOME . '/WordPress/core' );
define( 'WP_CONTENT_DIR', dirname(__FILE__) . '/content' );
define( 'WP_CONTENT_URL', WP_HOME . '/WordPress/content' );
```
* Copy the _index.php_ file (`cp core/index.php ..`) and change the location of the _wp-blog-header.php_ to `require( dirname( __FILE__ ) . '/WordPress/core/wp-blog-header.php' );`
* Create following directories to separate the individual files from the core installation
  * `mkdir -p config content/plugins content/themes content/uploads`

####  Follow some security instructions
(see book _Building Web Apps with WordPress_ / 78-1-4493-6407-6, chapter 8: Secure WordPress)
  * Use your own _table_prefix_
  * Disallow admins to edit plugins or themes using the admin interface: `define( 'DISALLOW_FILE_EDIT', true );`
  * Setup some specific functions for your general site theme (file: `content/themes/hvboom/functions.php`)

```PHP
<?php
/**
 * Obscure, if the username or the password was wrong
 */
add_filter( 'login_errors',
  create_function( '$a', '"Invalid username or password.";' )
);

/**
 * Don't write the WordPress version into generated files
 */
add_filter( 'the_generator', '__return_null' );

/**
 * Hide standard login url
 */
function hvboom_wp_login_filter( $url, $path, $orig_scheme ) {
    $old = array( "/(wp-login\.php)/" );
    $new = array( "members" );
        return preg_replace( $old, $new, $url, 1 );
}
add_filter( 'site_url', 'hvboom_wp_login_filter', 10, 3 );

function hvboom_wp_login_redirect() {
        if ( strpos( $_SERVER["REQUEST_URI"], 'members' ) === false ) {
                wp_redirect( site_url() );
                exit();
        }
}
add_action( 'login_init', 'hvboom_wp_login_redirect' );
?>
```

##  Create a site wide standard scheme

### Setup [Sass](http://sass-lang.com) support on [SwitchPlus](https://www.switchplus.ch)
* Using SSH to login to your account
* Install latest [RVM](https://rvm.io) (Ruby Version Manager) version
  ```Bash
  $ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
  $ \curl -sSL https://get.rvm.io | bash
  $ rvm --version
    rvm 1.26.11 (master) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
  ```
* Install current ruby version
  `rvm autolibs disable` is necessary, because otherwise RVM tries to install dependencies using the `sudo` command, which is not available on SwitchPlus
  ```Bash
  $ rvm autolibs disable
  $ rvm install 2.2
  $ ruby --version
    ruby 2.2.2p95 (2015-04-13 revision 50295) [x86_64-linux]
  ```
* Install [Sass](http://sass-lang.com/install) gem
  ```Bash
  $ gem install sass
  $ sass --version
    Sass 3.4.13 (Selective Steve)
  ```

## Useful plugins / themes
* [Font Awesome 4 Menus](http://www.newnine.com/plugins/font-awesome-4-menus)
* [Google Maps Widget](http://www.googlemapswidget.com/)
* [NextGEN gallery](http://www.nextgen-gallery.com/)
* [Responsive Lightbox](http://www.dfactory.eu/plugins/responsive-lightbox/)
* [TablePress](https://tablepress.org/)
* [WordPress PopUp](http://premium.wpmudev.org/project/the-pop-over-plugin/)
