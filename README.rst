Speciality Distribution for TYPO3 CMS
=====================================

Motivation
----------

All started with the modernisation of our Dummy package we were using in our company. To give a bit of background, we were aiming to:

* Have Twitter Bootstrap as HTML / CSS Framework
* Use as much as possible Fluid for the rendering and the templating. Actually, it turned out we reached the 100% thanks to the work of Claus Due

We wanted not only a package to demonstrate the capability of TYPO3 but also something useful so that it should save us from the tedious and repeating work when kick-starting a website. The result is pretty much promising. More important we **have put everything in public** so that you can test and also take advantage for your own needs.


Umbrella repository
-------------------

The Composer install will install all the necessary extension for you, among other:

* extension `speciality`_ which contains the template and configuration files for the website. Please report and make Pull Request **in this repository**.
* extension `speciality_distribution`_ which hosts the initial SQL dump of the distribution along with some demo images.

This extension is only required once. Before going live, it is advised to complete remove the extension which is not necessary anymore.

.. _speciality: https://github.com/Ecodev/speciality
.. _speciality_distribution: https://github.com/Ecodev/speciality_distribution

Installation
------------

Use Composer to install the Speciality Distribution. Notice the `system requirement`_ before proceeding and make sure PHP 5.5.x
and MariaDB / MySQL 5.1.x-5.5.x is installed on your system::

	# Download the Speciality Distribution via Composer
	composer create-project ecodev/typo3-cms-speciality-distribution domain.tld dev-master --no-dev

Once done, here are the usually steps I do on my local or on the remote server via SSH:

::

	# Prepare installation
	cd domain.tld
	touch htdocs/FIRST_INSTALL

	# Manual steps
	-> configure a Virtual Host.
	-> add a DNS entry (e.g editing /etc/hosts file)

	open http://domain.tld

	# Open the Backend at the end of the Wizard.
	http://domain.tld/typo3

	# Install the extension containing the distribution
	-> Open the Extension Manager
	-> Firstly, deactivate EXT:fluid_styled_content, EXT:context_help, EXT:func, ...
	-> Search for extension "speciality_distribution" and install it. It can take a while!
	-> At the end of the installation, reload the whole Backend and :)

.. _system requirement: https://github.com/TYPO3/TYPO3.CMS/blob/master/INSTALL.md

Support
=======

Bugs and wishes can be reported on the `bug tracker`_. You can also take advantage of some commercial support related to the Bootstrap Package by contacting contact@ecodev.ch.

.. _bug tracker: https://github.com/Ecodev/bootstrap_package/issues

How to continue?
================

As a next step, you likely want to change the CSS, add some custom layouts or customize configuration.
The place to head to is ``EXT:speciality`` which is located at ``htdocs/typo3conf/ext/speciality``. The name "speciality"
is just the extension key we are using in our company as convention. We keep it across our projects so that we don't have to think more
where to find the source code. This is not a big deal to change the name in case. However, the extension is mandatory and contains:

* HTML templates - ``EXT:speciality/Resources/Private/``
* Public resources such as JavaScript and CSS files  - ``EXT:speciality/Resources/Public/``
* PHP Code - ``EXT:speciality/Classes/``

For more detail, refer to https://github.com/Ecodev/speciality

Internal steps
==============

These are internal steps that I follow when bootstrapping a new website.

* Deactivate "speciality_distribution"
* Edit file ``.gitignore`` and add all files to the git repository.

::

	cd domain.tld
	nano .gitignore

	Remove:

	/htdocs/typo3conf/LocalConfiguration.php
	/htdocs/typo3conf/PackageStates.php

    Add:

	!/htdocs/typo3conf/ext/speciality

* Extension speciality under Git::

    git add .
    rm -rf htdocs/typo3conf/ext/speciality/.git
    rm htdocs/typo3conf/ext/speciality/{composer.json,README.md}

    # Temporary move
    mv htdocs/typo3conf/ext/speciality htdocs/typo3conf/ext/speciality2

    # Include speciality into the repository
    git rm htdocs/typo3conf/ext/speciality

    # Move back
    mv htdocs/typo3conf/ext/speciality2 htdocs/typo3conf/ext/speciality

    # Git finish the work
    git add .
    git ci -m "First commit"

* Add extension https://github.com/Ecodev/speciality_ecodev, edit composer.json at the root::

        {
                "type": "git",
                "url": "https://github.com/Ecodev/speciality_ecodev.git"
        }

        "require": {
            "ecodev/speciality-ecodev": "dev-master"
        },

* Remove "speciality_distribution" and "speciality" from composer.json
* ``composer update``
* ``git checkout htdocs/typo3conf/ext/speciality``
* Activate "speciality_ecodev" in EM.
* Change password of admin user
* Run following SQL request::

    DELETE FROM be_users WHERE username = 'editor' OR username = 'powereditor';
    TRUNCATE TABLE sys_news;
    TRUNCATE TABLE backend_layout;
    TRUNCATE TABLE pages_language_overlay;
    TRUNCATE TABLE sys_category;
    TRUNCATE TABLE sys_category_record_mm;
    TRUNCATE TABLE fe_groups;
    TRUNCATE TABLE fe_users;
    TRUNCATE TABLE sys_file;
    TRUNCATE TABLE sys_file_metadata;
    TRUNCATE TABLE sys_file_reference;
    DELETE FROM sys_file_storage WHERE uid != 1;

* Remove the DB credentials from ``htdocs/typo3conf/LocalConfiguration.php`` and add them into ``configuration/Settings.php``
* Edit ``htdocs/typo3conf/AdditionalConfiguration.php``::

    <?php
    // Include global configuration.
    require_once (PATH_site . '../configuration/Settings.php');
    require_once (PATH_site . 'typo3conf/ext/speciality_ecodev/Configuration/PHP/Typo3ConfVars.php');

    # Frontend Settings
    $GLOBALS['TYPO3_CONF_VARS']['FE']['pageNotFound_handling'] = '/404/';
    $GLOBALS['TYPO3_CONF_VARS']['FE']['pageUnavailable_handling'] = '/503/';
    // Declare Content Rendering for the Frontend
    $GLOBALS['TYPO3_CONF_VARS']['FE']['contentRenderingTemplates'] = array('fluidcontentcore/Configuration/TypoScript/');
    $GLOBALS['TYPO3_CONF_VARS']['FE']['activateContentAdapter'] = 0;


* In the BE again, configure RealURL in EM ``typo3conf/ext/speciality/Configuration/RealUrl/Configuration.php``
* In the BE again, add a sys_domain record on the root page and test http://domain.tld/sitemap.xml
* Add at the end of ``htdocs/robots.txt``::

    Sitemap: http://www.domain.tld/sitemap.xml

* ``rm -rf htdocs/fileadmin/speciality_distribution htdocs/fileadmin/_temp_``
* ``nano htdocs/typo3conf/LocalConfiguration.php``
    * under "SYS", add ``'trustedHostsPattern' => '.*\\.domain\\.tld|lan:*',``
    * update key ``sitename``
* ``git add . ; git ci --amend`` and a final ``git push``
* check CRON configuration and polish Scheduler tasks
