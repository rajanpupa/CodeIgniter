##############
Session Driver
##############

The Session class permits you maintain a user's "state" and track their
activity while they browse your site. CodeIgniter offers two default
session drivers: the classic `Cookie Driver`_, and the `Native Driver`_,
which supports usage of the native PHP Session mechanism. In addition,
you may create your own `Custom Drivers`_ to store session data however
you wish, while still taking advantage of the features of the Session class.

Initializing a Session
======================

Sessions will typically run globally with each page load, so the session
class must either be :doc:`initialized <../general/drivers>` in your
:doc:`controller <../general/controllers>` constructors, or it can be
:doc:`auto-loaded <../general/autoloader>` by the system. For the most
part the session class will run unattended in the background, so simply
initializing the class will cause it to read, create, and update
sessions.

To initialize the Session class manually in your controller constructor,
use the $this->load->driver function::

	$this->load->driver('session');

Once loaded, the Sessions library object will be available using:
$this->session

.. note:: For backward compatibility, the Session class may stil be loaded
	using the $this->load->library function, but converting your applications
	to use $this->load->driver is strongly recommended.

How do Sessions work?
=====================

When a page is loaded, the session class will check to see if valid
session data exists in the user's session. If sessions data does **not**
exist (or if it has expired) a new session will be created and saved.
If a session does exist, its information will be updated. With each update,
the session_id will be regenerated.

It's important for you to understand that once initialized, the Session
class runs automatically. There is nothing you need to do to cause the
above behavior to happen. You can, as you'll see below, work with
session data or even add your own data to a user's session, but the
process of reading, writing, and updating a session is automatic.

What is Session Data?
=====================

A *session*, as far as CodeIgniter is concerned, is simply an array
containing the following information:

-  The user's unique Session ID (this is a statistically random string
   with very strong entropy, hashed with MD5 for portability, and
   regenerated (by default) every five minutes)
-  The user's IP Address
-  The user's User Agent data (the first 120 characters of the browser
   data string)
-  The "last activity" time stamp.

The above data is stored in a cookie as a serialized array with this
prototype::

	[array]
	(
	     'session_id'    => random hash,
	     'ip_address'    => 'string - user IP address',
	     'user_agent'    => 'string - user agent data',
	     'last_activity' => timestamp
	)

.. note:: Sessions are only updated every five minutes by default to
	reduce processor load. If you repeatedly reload a page you'll notice
	that the "last activity" time only updates if five minutes or more has
	passed since the last time the cookie was written. This time is
	configurable by changing the $config['sess_time_to_update'] line in
	your system/config/config.php file.

Retrieving Session Data
=======================

Any piece of information from the session array is available using the
following function::

	$this->session->userdata('item');

Where item is the array index corresponding to the item you wish to
fetch. For example, to fetch the session ID you will do this::

	$session_id = $this->session->userdata('session_id');

.. note:: The function returns NULL if the item you are
	trying to access does not exist.

Adding Custom Session Data
==========================

A useful aspect of the session array is that you can add your own data
to it and it will be stored in the user's cookie. Why would you want to
do this? Here's one example:

Let's say a particular user logs into your site. Once authenticated, you
could add their username and email address to the session, making
that data globally available to you without having to run a database
query when you need it.

To add your data to the session array involves passing an array
containing your new data to this function::

	$this->session->set_userdata($array);

Where $array is an associative array containing your new data. Here's an
example::

	$newdata = array(
	                   'username'  => 'johndoe',
	                   'email'     => 'johndoe@some-site.com',
	                   'logged_in' => TRUE
	               );

	$this->session->set_userdata($newdata);

If you want to add userdata one value at a time, set_userdata() also
supports this syntax.

::

	$this->session->set_userdata('some_name', 'some_value');

If you want to verify that a userdata value exists, call has_userdata().

::

	$this->session->has_userdata('some_name');

Retrieving All Session Data
===========================

An array of all userdata can be retrieved as follows::

	$this->session->all_userdata()

And returns an associative array like the following::

	Array
	(
	    [session_id] => 4a5a5dca22728fb0a84364eeb405b601
	    [ip_address] => 127.0.0.1
	    [user_agent] => Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_7;
	    [last_activity] => 1303142623
	)

Removing Session Data
=====================

Just as set_userdata() can be used to add information into a session,
unset_userdata() can be used to remove it, by passing the session key.
For example, if you wanted to remove 'some_name' from your session
information::

	$this->session->unset_userdata('some_name');


This function can also be passed an associative array of items to unset.

::

	$array_items = array('username' => '', 'email' => '');

	$this->session->unset_userdata($array_items);


Flashdata
=========

CodeIgniter supports "flashdata", or session data that will only be
available for the next server request, and are then automatically
cleared. These can be very useful, and are typically used for
informational or status messages (for example: "record 2 deleted").

.. note:: Flash variables are prefaced with "flash\_" so avoid this prefix
	in your own session names.

To add flashdata::

	$this->session->set_flashdata('item', 'value');


You can also pass an array to set_flashdata(), in the same manner as
set_userdata().

To read a flashdata variable::

	$this->session->flashdata('item');
	
An array of all flashdata can be retrieved as follows::

	$this->session->all_flashdata();


If you find that you need to preserve a flashdata variable through an
additional request, you can do so using the keep_flashdata() function.

::

	$this->session->keep_flashdata('item');

Tempdata
========

CodeIgniter also supports "tempdata", or session data with a specific
expiration time. After the value expires, or the session expires or is
deleted, the value is automatically removed.

To add tempdata::

	$expire = 300;	// Expire in 5 minutes

	$this->session->set_tempdata('item', 'value', $expire);

You can also pass an array to set_tempdata()::

	$tempdata = array('newuser' => TRUE, 'message' => 'Thanks for joining!');

	$this->session->set_tempdata($tempdata, '', $expire);

.. note:: If the expiration is omitted or set to 0, the default expiration of
	5 minutes will be used.

To read a tempdata variable::

	$this->session->tempdata('item');

If you need to remove a tempdata value before it expires,
use unset_tempdata()::

	$this->session->unset_tempdata('item');

Destroying a Session
====================

To clear the current session::

	$this->session->sess_destroy();

.. note:: This function should be the last one called, and even flash
	variables will no longer be available. If you only want some items
	destroyed and not all, use unset_userdata().

Session Preferences
===================

You'll find the following Session related preferences in your
application/config/config.php file:

=========================== =============== =========================== ==========================================================================
Preference                  Default         Options                     Description
=========================== =============== =========================== ==========================================================================
**sess_driver**             cookie          cookie/native/*custom*      The initial session driver to load.
**sess_valid_drivers**      cookie, native  None                        Additional valid drivers which may be loaded.
**sess_cookie_name**        ci_session      None                        The name you want the session cookie saved as (data for Cookie driver or
                                                                        session ID for Native driver).
**sess_expiration**         7200            None                        The number of seconds you would like the session to last. The default
                                                                        value is 2 hours (7200 seconds). If you would like a non-expiring
                                                                        session set the value to zero: 0
**sess_expire_on_close**    FALSE           TRUE/FALSE (boolean)        Whether to cause the session to expire automatically when the browser
                                                                        window is closed.
**sess_encrypt_cookie**     FALSE           TRUE/FALSE (boolean)        Whether to encrypt the session data (Cookie driver only).
**sess_use_database**       FALSE           TRUE/FALSE (boolean)        Whether to save the session data to a database. You must create the
                                                                        table before enabling this option (Cookie driver only).
**sess_table_name**         ci_sessions     Any valid SQL table name    The name of the session database table (Cookie driver only).
**sess_time_to_update**     300             Time in seconds             This options controls how often the session class will regenerate itself
                                                                        and create a new session id.
**sess_match_ip**           FALSE           TRUE/FALSE (boolean)        Whether to match the user's IP address when reading the session data.
                                                                        Note that some ISPs dynamically changes the IP, so if you want a
                                                                        non-expiring session you will likely set this to FALSE.
**sess_match_useragent**    TRUE            TRUE/FALSE (boolean)        Whether to match the User Agent when reading the session data.
=========================== =============== =========================== ==========================================================================

In addition to the values above, the cookie and native drivers apply the
following configuration values shared by the :doc:`Input <input>` and
:doc:`Security <security>` classes:

=========================== =============== ==========================================================================
Preference                  Default         Description
=========================== =============== ==========================================================================
**cookie_prefix**           ''              Set a cookie name prefix in order to avoid name collisions
**cookie_domain**           ''              The domain for which the session is applicable
**cookie_path**             /               The path to which the session is applicable
=========================== =============== ==========================================================================

Session Drivers
===============

By default, the `Cookie Driver`_ is loaded when a session is initialized.
However, any valid driver may be selected with the $config['sess_driver']
line in your config.php file.

The session driver library comes with the cookie and native drivers
installed, and `Custom Drivers`_ may also be installed by the user.

Typically, only one driver will be used at a time, but CodeIgniter does
support loading multiple drivers. If a specific valid driver is called, it
will be automatically loaded. Or, an additional driver may be explicitly
loaded by calling load_driver()::

	$this->session->load_driver('native');

The Session library keeps track of the most recently selected driver to call
for driver methods. Normally, session class methods are called directly on
the parent class, as illustrated above. However, any methods called through
a specific driver will select that driver before invoking the parent method.

So, alternation between multiple drivers can be achieved by specifying which
driver to use for each call::

	$this->session->native->set_userdata('foo', 'bar');

	$this->session->cookie->userdata('foo');

	$this->session->native->unset_userdata('foo');

Notice in the previous example that the *native* userdata value 'foo'
would be set to 'bar', which would NOT be returned by the call for
the *cookie* userdata 'foo', nor would the *cookie* value be unset by
the call to unset the *native* 'foo' value. The drivers maintain independent
sets of values, regardless of key names.

A specific driver may also be explicitly selected for use by pursuant
methods with the select_driver() call::

	$this->session->select_driver('native');

	$this->session->userdata('item');	// Uses the native driver

Cookie Driver
-------------

The Cookie driver stores session information for each user as serialized
(and optionally encrypted) data in a cookie. It can also store the session
data in a database table for added security, as this permits the session ID
in the user's cookie to be matched against the stored session ID. By default
only the cookie is saved. If you choose to use the database option you'll
need to create the session table as indicated below.

If you have the encryption option enabled, the serialized array will be
encrypted before being stored in the cookie, making the data highly
secure and impervious to being read or altered by someone. More info
regarding encryption can be :doc:`found here <encryption>`, although
the Session class will take care of initializing and encrypting the data
automatically.

.. note:: Even if you are not using encrypted sessions, you must set
	an :doc:`encryption key <./encryption>` in your config file which is used
	to aid in preventing session data manipulation.

.. note:: Cookies can only hold 4KB of data, so be careful not to exceed
	the capacity. The encryption process in particular produces a longer
	data string than the original so keep careful track of how much data you
	are storing.

Saving Session Data to a Database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While the session data array stored in the user's cookie contains a
Session ID, unless you store session data in a database there is no way
to validate it. For some applications that require little or no
security, session ID validation may not be needed, but if your
application requires security, validation is mandatory. Otherwise, an
old session could be restored by a user modifying their cookies.

When session data is available in a database, every time a valid session
is found in the user's cookie, a database query is performed to match
it. If the session ID does not match, the session is destroyed. Session
IDs can never be updated, they can only be generated when a new session
is created.

In order to store sessions, you must first create a database table for
this purpose. Here is the basic prototype (for MySQL) required by the
session class::

	CREATE TABLE IF NOT EXISTS  `ci_sessions` (
		session_id varchar(40) DEFAULT '0' NOT NULL,
		ip_address varchar(45) DEFAULT '0' NOT NULL,
		user_agent varchar(120) NOT NULL,
		last_activity int(10) unsigned DEFAULT 0 NOT NULL,
		user_data text NOT NULL,
		PRIMARY KEY (session_id, ip_address, user_agent),
		KEY `last_activity_idx` (`last_activity`)
	);

.. note:: By default the table is called ci_sessions, but you can name
	it anything you want as long as you update the
	application/config/config.php file so that it contains the name you have
	chosen. Once you have created your database table you can enable the
	database option in your config.php file as follows::

		$config['sess_use_database'] = TRUE;

	Once enabled, the Session class will store session data in the DB.

	Make sure you've specified the table name in your config file as well::

		$config['sess_table_name'] = 'ci_sessions';

.. note:: The Cookie driver has built-in garbage collection which clears
	out expired sessions so you do not need to write your own routine to do
	it.

Native Driver
-------------

The Native driver relies on native PHP sessions to store data in the
$_SESSION superglobal array. All stored values continue to be available
through $_SESSION, but flash- and temp- data items carry special prefixes.

Custom Drivers
--------------

You may also :doc:`create your own <../general/creating_drivers>` custom
session drivers. A session driver basically manages an array of name/value
pairs with some sort of storage mechanism.

To make a new driver, extend CI_Session_driver. Overload the initialize()
method and read or create session data. Then implement a save handler to
write changed data to storage (sess_save), a destroy handler to remove
deleted data (sess_destroy), a regenerate handler to make a new session ID
(sess_regenerate), and an access handler to expose the data (get_userdata).
Your initial class might look like::

	class CI_Session_custom extends CI_Session_driver {
		protected function initialize()
		{
			// Read existing session data or create a new one
		}

		public function sess_save()
		{
			// Save current data to storage
		}

		public function sess_destroy()
		{
			// Destroy the current session and clean up storage
		}

		public function sess_regenerate()
		{
			// Create new session ID
		}

		public function &get_userdata()
		{
			// Return a reference to your userdata array
		}
	}

Notice that get_userdata() returns a reference so the parent library is
accessing the same array the driver object is using. This saves memory
and avoids synchronization issues during usage.

Put your driver in the libraries/Session/drivers folder anywhere in your
package paths. This includes the application directory, the system directory,
or any path you add with $CI->load->add_package_path(). Your driver must be
named CI_Session_<name>, and your filename must be Session_<name>.php,
preferably also capitalized, such as::

	CI_Session_foo in libraries/Session/drivers/Session_foo.php

Then specify the driver by setting 'sess_driver' in your config.php file or as a
parameter when loading the CI_Session object::

	$config['sess_driver'] = 'foo';

OR::

	$CI->load->driver('session', array('sess_driver' => 'foo'));

The driver specified by 'sess_driver' is automatically included as a valid
driver. However, if you want to make a custom driver available as an option
without making it the initially loaded driver, set 'sess_valid_drivers' in
your config.php file to an array including your driver name::

	$config['sess_valid_drivers'] = array('sess_driver');

