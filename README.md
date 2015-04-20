# DESCRIPTION

This cookbook install zabbix-agent and zabbix-server.

By defaut the cookbook installs zabbix-agent, check the attribute for enable/disable zabbix_server / web or disable zabbix_agent installation.

Default login password for zabbix frontend is admin / zabbix  CHANGE IT !

# USAGE

Be careful when you update your server version, you need to run the sql patch in /opt/zabbix-$VERSION.

If you do not specify source\_url attributes for agent or server they will be set to download the specified
branch and version from the official Zabbix source repository. If you want to upgrade later, you need to
either nil out the source\_url attributes or set them to the url you wish to download from.

    node['zabbix']['agent']['source_url'] = nil
    node['zabbix']['server']['source_url'] = nil

Please include the default recipe before using any other recipe.

Installing the Agent :

    "recipe[zabbix]"

Installing the Server :

    "recipe[zabbix]",  
    "recipe[zabbix::server]"

Installing the Database :

    "recipe[mysql::server]",
    "recipe[zabbix]",
    "recipe[zabbix::database]"

Installing all 3 - Database MUST come before Server

    "recipe[database::mysql]",
    "recipe[mysql::server]",
    "recipe[zabbix]",
    "recipe[zabbix::database]",
    "recipe[zabbix::server]"

NOTE:

If you are running on Redhat, Centos, Scientific or Amazon, you will need packages from EPEL.
Include "recipe[yum::epel]" in your runlist or satisfy these requirements some other way.

    "recipe[yum::epel]"

# ATTRIBUTES

Don't forget to set :

    node.set['zabbix']['agent']['servers'] = ["Your_zabbix_server.com","secondaryserver.com"]
    node.set['zabbix']['web']['fqdn'] or you will not have the zabbix web interface

Note :

A Zabbix agent running on the Zabbix server will need to :

* use a different account than the on the server uses or it will be able to spy on private data.
* specify the local Zabbix server using the localhost (127.0.0.1, ::1) address.

example :

## Server

	  node.set['zabbix']['server']['branch'] = "ZABBIX%20Latest%20Stable"
	  node.set['zabbix']['server']['version'] = "2.0.0"
	  node.set['zabbix']['server']['source_url'] = nil
	  ndoe.set['zabbix']['server']['install_method'] = "source"

## Agent

	  node.set['zabbix']['agent']['branch'] = "ZABBIX%20Latest%20Stable"
	  node.set['zabbix']['agent']['version'] = "2.0.0"
	  node.set['zabbix']['agent']['source_url'] = nil
	  node.set['zabbix']['agent']['install_method'] = "prebuild"

## Database

    node.set['zabbix']['database']['install_method'] = 'mysql'
    node.set['zabbix']['database']['dbname'] = "zabbix"
    node.set['zabbix']['database']['dbuser'] = "zabbix"
    node.set['zabbix']['database']['dbhost'] = "localhost"
    node.set['zabbix']['database']['dbpassword'] = 'password'
    node.set['zabbix']['database']['dbport'] = "3306"

If you are using AWS RDS

    node.set['zabbix']['database']['install_method'] = 'rds_mysql'
    node.set['zabbix']['database']['rds_master_user'] = 'username'
    node.set['zabbix']['database']['rds_master_password'] = 'password'



# RECIPES

## default

The default recipe creates the Zabbix user and directories used by all Zabbix components.

Optionally, it installs the Zabbix agent.

You can control the agent install with the following attributes:

    node['zabbix']['agent']['install'] = true
    node['zabbix']['agent']['install_method'] = 'source'

## agent\_prebuild

Downloads and installs the Zabbix agent from a pre built package

If you are on a machine in the RHEL family of platforms, then you must have your
package manager setup to allow installation of:

    package "redhat-lsb"

You can control the agent version with:

    node['zabbix']['agent']['version']

## agent\_source

Downloads and installs the Zabbix agent from source

If you are on a machine in the RHEL family of platforms, then you will
need to install packages from the EPEL repository. The easiest way to do this
is to add the following recipe to your runlist before zabbix::agent\_source

    recipe "yum::epel"

You can control the agent install with:

    node['zabbix']['agent']['branch']
    node['zabbix']['agent']['version']
    node['zabbix']['agent']['configure_options']

## database

WARNING: This recipe persists your database credentials back to the Chef server
as plaintext  node attributes. To prevent this, consume the `zabbix_database`
LWRP in your own wrapper cookbook.

Creates and initializes the Zabbix database

Currenly only supports MySql and RDS MySql databases

If they are not already set, this recipe will generate the following attributes:

    node['zabbix']['database']['dbpassword']
    node['mysql']['server_root_password'] # Not generated if you are using RDS

You can control the database version with:

    node['zabbix']['server']['branch']
    node['zabbix']['server']['version']

The database setup uses the following attributes:

    node['zabbix']['database']['dbhost']
    node['zabbix']['database']['dbname']
    node['zabbix']['database']['dbuser']
    node['zabbix']['database']['dbpassword']

    node['zabbix']['database']['install_method']

If `install_method` is 'mysql' you also need:

    node['mysql']['server_root_password']

If `install_method` is 'rds\_mysql' you also need:

    node['zabbix']['database']['rds_master_username']
    node['zabbix']['database']['rds_master_password']

## firewall

Opens firewall rules to allow Zabbix nodes to communicate with each other.

## server

Delegates to other recipes to install the Zabbix server and Web components.

You can control the server and web installs with the following attributes:

    node['zabbix']['server']['install'] = true
    node['zabbix']['server']['install_method'] = 'source'
    node['zabbix']['web']['install'] = true

If you are using a MySql or RDS MySql database make sure your runlist
includes:

    "recipe[database::mysql]",
    "recipe[mysql::client]"

If you are user a Postgres database make sure your runlist includes:

    "recipe[database::postgresql]",
    "recipe[postgresql::client]",

## server\_source

Downloads and installs the Zabbix Server component from source

If you are on a machine in the RHEL family of platforms, then you will
need to install packages from the EPEL repository. The easiest way to do this
is to add the following recipe to your runlist before zabbix::server\_source

    recipe "yum::epel"

You can control the server install with:

    node['zabbix']['server']['branch']
    node['zabbix']['server']['version']
    node['zabbix']['server']['configure_options']

The server also needs to know about:

    node['zabbix']['database']['dbhost']
    node['zabbix']['database']['dbname']
    node['zabbix']['database']['dbuser']
    node['zabbix']['database']['dbpassword']
    node['zabbix']['database']['dbport']

## web

Creates an Apache site for the Zabbix Web component

# LWRPs

## database

### resources/database

Installs the Zabbix Database

The default provider is Chef::Provider::ZabbixDatabaseMySql in "providers/database_my_sql".
If you want a different provider, make sure you set the following in your resource call.

    provider Chef::Provider::SomeProviderClass

#### Actions

* `create` (Default Action) - Creates the Zabbix Database

#### Attributes

* `dbname` (Name Attribute) -  Name of the Zabbix databse to create
* `host` - Host to create the database on
* `port` - Port to connext to the database over
* `username` - Name of the Zabbix database user
* `password` - Password for the Zabbix database user
* `root_username` - Name of the root user for the database server
* `root_password` - Password for the database root user
* `allowed_user_hosts` (Default: '') - Where users can connect to the database from
* `server_branch` - Which branch of server code you are using
* `server_version` - Which version of server code you are using
* `source_dir` - Where Zabbix source code should be stored on the host
* `install_dir` - Where Zabbix should be installed to

### providers/database\_my\_sql

Installs a MySql or RDS MySql Zabbix Database

This is the default provider for `resources/database`

If you are using MySQL make sure you set

    root_username "root"
    root_password "your root password"

If you are using RDS MySql make sure you set

    root_username "your rds master username"
    root_password "your rds master password"

### providers/database\_postgres

Installs a Postgres Zabbix Database

Call the `zabbix_database` resource with

    provider Chef::Provider::ZabbixDatabasePostgres

Make sure you set

    root_username 'postgres'
    root_pasword  'your postgres admin password'

The `allowed_user_hosts` attribute is ignored

### resources/source

Fetchs the Zabbix source tar and does something with it

#### Actions
* `extract_only` (Default Action) - Just fetch and extract the tar
* `install_server` - Fetch the tar then compile the source as a Server
* `install_agent` - Fetch the tar then compile the source as an Agent

#### Attributes
* `name` (Name Attribute) - An arbitrary name for the resource
* `branch` - The branch of Zabbix to grab code for
* `version` - The version of Zabbix to grab code for
* `code_dir` - Where Zabbix source code should be stored on the host
* `target_dir` - A sub directory under `code_dir` where you want the source extracted
* `install_dir` (Optional) - Where Zabbix should be installed to
* `configure_options` (Optional) - Flags to use when compiling Zabbix

### providers/source:

Default implementation of how to Fetch and handle the Zabbix source code.


# TODO

* Support more platform on agent side windows ?
* LWRP Magic ?
