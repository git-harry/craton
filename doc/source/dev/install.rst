
=====================================================
Installing and Setting up a Development Environment
=====================================================

Installation
============

.. note:: *This is a Python3 project.*

.. note:: *This project requires MySQL 5.7, until a stable release of MariaDB with JSON function support is available*

---------------------
Ubuntu 16.04 (Xenial)
---------------------


* Install a fresh Ubuntu image

* Make sure we have git installed::

    # apt-get update
    # apt-get install git -y

* Clone the repository::

    # git clone https://github.com/openstack/craton.git

* Install the prerequisite packages::

    # apt-get install python3.5 python3.5-dev
    # apt-get install python3-pip python3-setuptools
    # python3 -m pip install --upgrade pip setuptools

* Goto craton directory and install the following::

    # python3 -m pip install -r requirements.txt
    # python3 -m pip install .

* Install mysql-server and make sure mysql is running::

    # apt-get install mysql-server-5.7 mysql-client-5.7
    # systemctl enable mysql
    # systemctl start mysql

* Ensure you have python3-mysqldb installed::

    # apt-get install python3-mysqldb

--------
CentOS 7
--------


* Install a fresh CentOS 7 image

* Make sure we have git installed::

    # yum update
    # yum install git -y

* Clone the repository::

    # git clone https://github.com/openstack/craton.git

* Install the prerequisite packages::

    # yum install python34-devel python34-pip python34-setuptools gcc
    # python3 -m pip install --upgrade pip setuptools

* Goto craton directory and install the following::

    # python3 -m pip install -r requirements.txt
    # python3 -m pip install .

* Install mysql-server community release from `MySQL Community Page`_::

    # wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
    # rpm -ivh mysql57-community-release-el7-9.noarch.rpm
    # yum install mysql-server
    # systemctl enable mysqld
    # systemctl start mysqld

* Ensure you have MySQL-python installed::

    # yum install MySQL-python

* Setup Database User and secure installation::

    # grep 'temporary password' /var/log/mysqld.log
    # mysql_secure_installation

---------
Fedora 25
---------


* Install a fresh Fedora 25 image

* Make sure we have git installed::

    # dnf update
    # dnf install git -y

* Clone the repository::

    # git clone https://github.com/openstack/craton.git

* Install the prerequisite packages::

    # dnf install python3-devel python3-pip python3-setuptools gcc redhat-rpm-config
    # python3 -m pip install --upgrade pip setuptools

* Goto craton directory and install the following::

    # python3 -m pip install -r requirements.txt
    # python3 -m pip install .

* Install mysql-server and make sure mysql is running::

    # dnf install mysql-server
    # systemctl enable mysqld
    # systemctl start mysqld

* Ensure you have python3-mysql installed::

    # dnf install python3-mysql

--------------
Database Setup
--------------

* Connect to database server as root user::

    # mysql -u root -p

* Create user craton::

    # CREATE USER 'craton'@'localhost' IDENTIFIED BY 'craton';

* Grant proper access to the craton user and flush privileges::

    # GRANT ALL PRIVILEGES ON craton.* TO 'craton'@'localhost'
    identified by 'craton';
    # FLUSH PRIVILEGES;

* You can verify that the user was added by calling::

    # select host, user, password from mysql.user;

* Create the Craton database::

    # create database craton CHARACTER SET='utf8';

* Logout from the database server::

    # exit

------------------------------------
Create etc/craton-api-conf.dev
------------------------------------
* Copy the sample config in the etc directory to make a development config file.

    # cp craton-api-conf.sample craton-api-conf.dev

* Make api_paste_config use a fully qualified path (not relative).
  This will be specific for your machine

.. note:: Make sure you have the proper path for craton-api-conf.dev

    # api_paste_config=/home/cratonuser/craton/etc/craton-api-paste.ini

* Add the following line to the [database] section:

    # connection = mysql+pymysql://craton:craton@localhost/craton

* Update the host in the [api] section to match your IP:

    # host = xxx.xxx.xxx.xxx

----------
Run dbsync
----------

* Make sure to run dbsync to get the db tables created::

    # craton-dbsync --config-file=etc/craton-api-conf.dev version
    # craton-dbsync --config-file=etc/craton-api-conf.dev upgrade

* Make sure to run dbsync bootstrap to create initial project and root user::
  # craton-dbsync --config-file=etc/craton-api-conf.dev bootstrap

  Note: The above command outputs user, project-id and API key to use with
  python-cratonclient to interact with craton server.

---------------------
Start the API Service
---------------------

* To start the API service, run the following command::

    # craton-api --config-file=etc/craton-api-conf.dev


* Some examples of API calls are as below:

---------------
Create a Region
---------------

* In order to create the region, export the IP address you set in
  /etc/craton-api-conf.dev::

    # export MY_IP=xxx.xxx.xxx.xxx

* Next create a cloud to which the region is associated to::

    # curl -i "http://${MY_IP}:7780/v1/clouds" \
           -d '{"name": "Cloud_Sample"}' \
           -H "Content-Type: application/json" \
           -H "X-Auth-Token: demo" \
           -H "X-Auth-User: demo" \
           -H "X-Auth-Project: 717e9a216e2d44e0bc848398563bda06"

* To create region, execute the following command::

    # curl -i "http://${MY_IP}:7780/v1/regions" \
           -d '{"name": "DFW", "cloud_id": 1}' \
           -H "Content-Type: application/json" \
           -H "X-Auth-Token: demo" \
           -H "X-Auth-User: demo" \
           -H "X-Auth-Project: 717e9a216e2d44e0bc848398563bda06"

------------------
Get created Region
------------------

* To get the created region, execute the following command::

    # curl -i "http://${MY_IP}:7780/v1/regions" \
           -H "Content-Type: application/json" \
           -H "X-Auth-Token: demo" \
           -H "X-Auth-User: demo" \
           -H "X-Auth-Project: 717e9a216e2d44e0bc848398563bda06"

--------------------------
Get all hosts for Region 1
--------------------------

* To get all hosts for region 1, execute the following command::

    # curl -i "http://${MY_IP}:7780/v1/hosts?region_id=1" \
           -H "Content-Type: application/json" \
           -H "X-Auth-Token: demo" \
           -H "X-Auth-User: demo" \
           -H "X-Auth-Project: 717e9a216e2d44e0bc848398563bda06"

---------------------
Get a particular host
---------------------

* To get a particular host, execute the following command::

    # curl -i "http://${MY_IP}:7780/v1/hosts/33" \
           -H "Content-Type: application/json" \
           -H "X-Auth-Token: demo" \
           -H "X-Auth-User: demo" \
           -H "X-Auth-Project: 717e9a216e2d44e0bc848398563bda06"

-----------------------
Using wrapper functions
-----------------------

Some wrapper functions have been included in craton/tools to quickly build, reload, populate, and query craton.

* To load the wrapper functions, run the following in the craton parent directory::

    # source tools/wrapper-functions.sh

* To start craton directly, run the following from the craton parent directory::

    # craton-direct-start

* The following environment variables must be exported for working with the craton API server::

  * CRATON_URL
  * OS_PROJECT_ID
  * OS_USERNAME
  * OS_PASSWORD

* You can search the logs for these values or run::

    # export eval $(craton-direct-env)

* Populate craton with fake data by running::

    # craton-fake-data

* Run API calls against craton with the following wrappers::
.. note:: *Requires the installation of httpie*

    # craton-post v1/regions name=HKG
    # craton-get v1/hosts
    # craton-put v1/hosts/3 device_type=container
    # craton-put v1/hosts/3/variables foo=47 bar:='["a", "b", "c"]'
    # craton-delete v1/hosts/4

-------------
Running Tests
-------------

* To run unit tests, execute the following command::

    # tox

* To run functional tests, execute the following command::

    # tox -e functional

.. _MySql Community Page:
   https://dev.mysql.com/downloads/repo/yum/
