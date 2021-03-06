=====================================================
Shibboleth Integration (Enterprise Subscription only)
=====================================================

Introduction
------------

The ownCloud Shibboleth user backend application integrates ownCloud with a 
Shibboleth Service Provider (SP) and allows operations in federated and 
single-sign-on infrastructures. Setting up Shibboleth has three steps:

1. Create the appropriate Apache configuration.
2. Enable the Shibboleth app.
3. Map Shibboleth environment variables to ownCloud attributes.

Currently supported installations are based on the `native Apache integration`_ 
. The individual configuration of the service provider is highly dependent on 
the operating system, as well as on the integration with the Identity 
Providers (IdP), and require case-by-case analysis and installation.

The ownCloud Desktop Client can interact with an 
ownCloud instance running inside a Shibboleth Service Provider by using built-in 
browser components for authentication against the IdP.

The regular ownCloud Android and iOS mobile apps do not work with Shibboleth.
However, customers who create 
:doc:`branded mobile apps with ownBrander 
<../enterprise_clients/creating_branded_apps>`
have the option to enable SAML authentication in ownBrander.

Enterprise customers also have the option to request a regular ownCloud 
mobile client built to use Shibboleth from their ownCloud account 
representatives.

The ownCloud desktop sync client and mobile apps store users' logins, so 
your users only need to enter their logins the first time they set up their 
accounts. These screenshots show what the user sees at account setup. Figure 1 
shows a test Shibboleth login screen from 
`Testshib.org <https://www.testshib.org/index.html>`_ on the ownCloud desktop 
sync client.

.. figure:: ../images/shib-gui1.png
   :alt: First client login screen.

   *figure 1: First login screen*
   
Then after going through the setup wizard, the desktop sync client displays the 
server and login information just like it does for any other ownCloud server 
connections.

.. figure:: ../images/shib-gui4.png
   :alt: The ownCloud client shows which server you are connected to.

   *figure 2: ownCloud client displays server information*
   
To your users, it doesn't look or behave differently on the desktop sync 
client, Android app, or iOS app from an ordinary ownCloud account setup. The 
only difference is the initial setup screen where they enter their account 
login.

Apache Configuration
--------------------

This is an example configuration as installed and operated on a Linux server 
running the Apache Web server. These configurations are highly operating system 
specific and require a high degree of customization.

The ownCloud instance itself is installed in ``/var/www/owncloud/``.  The 
following aliases are defined in an Apache virtual host directive:

.. code-block:: apache

	# non-Shibboleth access
	Alias /owncloud /var/www/owncloud/
	# for Shibboleth access
	Alias /oc-shib /var/www/owncloud/

Further Shibboleth specific configuration as defined in 
``/etc/apache2/conf.d/shib.conf``:

.. code-block:: apache

	#
	# Load the Shibboleth module.
	#
	LoadModule mod_shib /usr/lib64/shibboleth/mod_shib_22.so

	#
	# Ensures handler will be accessible.
	#
	<Location /Shibboleth.sso>
	  Satisfy Any
	  Allow from all
	</Location>

	#
	# Configure the module for content.
	#
	# Shibboleth is disabled for the following location to allow non 
	# shibboleth webdav access
	<Location ~ "/oc-shib/remote.php/nonshib-webdav">
	  Satisfy Any
	  Allow from all
	  AuthType None
	  Require all granted
	</Location>

	# Shibboleth is disabled for the following location to allow public link 
	# sharing
	<Location ~ 
	  "/oc-shib/(status.php$
	  |index.php/s/
	  |public.php$
	  |cron.php$
	  |core/img/
	  |index.php/apps/files_sharing/ajax/publicpreview.php$
	  |index.php/apps/files/ajax/upload.php$
	  |apps/files/templates/fileexists.html$
	  |index.php/apps/files/ajax/mimeicon.php$)">
	  Satisfy Any
	  Allow from all
	  AuthType None
	  Require all granted
	</Location>

	# Shibboleth is disabled for the following location to allow public gallery 
    # sharing
	<Location ~ 
         "/oc-shib/(apps/gallery/templates/slideshow.html$
         |index.php/apps/gallery/ajax/getimages.php	
         |index.php/apps/gallery/ajax/thumbnail.php
         |index.php/apps/gallery/ajax/image.php)">
	  Satisfy Any
	  Allow from all
	  AuthType None
	  Require all granted
	</Location>

	# Shibboleth is disabled for the following location to allow public link 
	# sharing
	<Location ~ "/oc-shib/.*\.css">
	  Satisfy Any
	  Allow from all
	  AuthType None
	  Require all granted
	</Location>

	# Shibboleth is disabled for the following location to allow public link 
	# sharing
	<Location ~ "/oc-shib/.*\.js">
	  Satisfy Any
	  Allow from all
	  AuthType None
	  Require all granted
	</Location>

	# Shibboleth is disabled for the following location to allow public link
	# sharing
	<Location ~ "/oc-shib/.*\.woff ">
	  Satisfy Any
	  Allow from all
	  AuthType None
	  Require all granted
	</Location>

	# Besides the exceptions above this location is now under control of
	# Shibboleth
	<Location /oc-shib>
	  AuthType shibboleth
	  ShibRequireSession On
	  ShibUseHeaders Off
	  ShibExportAssertion On
	  require valid-user
	</Location>

Enabling & Configurating the Shibboleth App
-------------------------------------------

You must enable the Shibboleth app on your Apps page, and then select the mode 
you want Shibboleth to operate in from the dropdown on your Admin page, either 
**Autoprovision Users** or **Single sign-on only**.

.. figure:: ../images/shib-gui5.png
   :alt: Shibboleth configuration screen.

   *figure 3: Enabling Shibboleth on the Admin page*	

In ownCloud 8.1 the Shibboleth variables were stored in 
``apps/user_shibboleth/config.php``. This file was overwritten on upgrades. In 
ownCloud 8.2 the variables are stored in the ownCloud database, so Shibboleth 
is now automatically upgradeable.

After installing and enabling the Shibboleth application there are four 
Shibboleth environment configuration variables to map to ownCloud user 
attributes.

.. figure:: ../images/shib-gui6.png
   :alt: Dropdowns for mapping Shibboleth environment configuration variables to ownCloud user attributes.

   *figure 4: Mapping Shibboleth environment configuration variables to ownCloud user attributes*

WebDAV Support
--------------

Users of standard WebDAV clients can use an alternative 
WebDAV Url, for example ``https://cloud.example.com/remote.php/nonshib-webdav/``
to log in with their username and password. The password is generated on the 
Personal settings page.

.. image:: ../images/shibboleth-personal.png

.. note:: In pure SSO mode the WebDAV password feature will not work, as we 
   have no way to store the WebDAV password. It does work in auto-provision 
   mode.

For provisioning purpose an OCS API has been added to revoke a generated 
password for a user:

Syntax: ``/v1/cloud/users/{userid}/non_shib_password``

* HTTP method: DELETE

Status codes:

* 100 - successful
* 998 - user unknown

Example:

.. code-block:: bash

	$ curl -X DELETE "https://cloud.example.com/ocs/v1.php/cloud/users/myself@testshib.org/non_shib_password" -u admin:admin 
	<?xml version="1.0"?>
	<ocs>
	 <meta>
	  <status>ok</status>
	  <statuscode>100</statuscode>
	  <message/>
	 </meta>
	 <data/>
	</ocs>


Known Limitations
-----------------

Encryption
----------

File encryption can not be used together with Shibboleth because the encryption 
requires the user's password to unlock the private encryption key. Due to the 
nature of Shibboleth the user's password is not known to the service provider. 
Currently, we have no solution to this limitation.

Other Login Mechanisms
----------------------

Shibboleth is not compatible with any other ownCloud user backend because the 
login process is handled outside of ownCloud.

You can allow other login mechanisms (e.g. LDAP or ownCloud native) by creating 
a second Apache virtual host configuration. This second location is not 
protected by Shibboleth, and you can use your other ownCloud login mechanisms.

Session Timeout
---------------

Session timeout on Shibboleth is controlled by the IdP. It is not possible to 
have a session length longer than the length controlled by the IdP. In extreme 
cases this could result in re-login on mobile clients and desktop clients every 
hour.

The session timeout can be overridden in the service provider, but this 
requires a source code change of the Apache Shibboleth module. A patch can be 
provided by the ownCloud support team.

UID Considerations and Windows Network Drive compatability
----------------------------------------------------------

When using ``user_shibboleth`` in single-sign on (SSO) only mode, together with 
``user_ldap``, both apps need to resolve to the same ``uid``. 
``user_shibboleth`` will do the authentication, and ``user_ldap`` will provide 
user details such as ``email`` and ``displayname``. In the case of Active 
Directory, multiple attributes can be used as the ``uid``. But they all have 
different implications to take into account.

Attributes
^^^^^^^^^^

**sAMAccountName**

* *Example:* jfd
* *Uniqueness:* Domain local, might change e.g. marriage
* *Other implications:* Works with ``windows_network_drive`` app

**userPrincipalName**

* *Example:* jfd@owncloud.com
* *Uniqueness:* Forest local, might change on eg. marriage
* *Other implications:* TODO check WND compatability
  
**objectSid**

* *Example:* S-1-5-21-2611707862-2219215769-354220275-1137
* *Uniqueness:* Domain local, changes when the user is moved to a new domain
* *Other implications:* Incompatible with ``windows_network_drive`` app

**sIDHistory**

* *Example:* Multi-value
* *Uniqueness:* Contains previous objectSIDs
* *Other implications:* Incompatible with ``windows_network_drive`` app

**objectGUID**

* *Example:* 47AB881D-0655-414D-982F-02998C905A28
* *Uniqueness:* Globally unique
* *Other implications:* Incompatible with ``windows_network_drive`` app

Keep in mind that ownCloud will derive the home folder from the ``uid``, unless 
a home folder naming rule is in place. The only truly stable attribute is the 
``objectGUID``, so that should be used. If not for the ``uid`` then at least as 
the home folder naming rule. The tradeoff here is that if you want to use 
``windows_network_drive`` you are bound to the ``sAMAccountName``, as that is 
used as the login.

Also be aware that using ``user_shibboleth`` in Autoprovisioning mode will not 
allow you to use SSO for your ``user_ldap`` users, because ``uid`` collisions 
will be detected by ``user_ldap``.

.. _native Apache integration: 
    https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPApacheConfig
.. _WebDAV and Shibboleth: 
    https://wiki.shibboleth.net/confluence/display/SHIB2/WebDAV
