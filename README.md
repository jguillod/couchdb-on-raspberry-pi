# How to install CouchDB on Raspbian Stretch / Raspberry PI #

> Also published on [hackster.io](https://www.hackster.io/mehealth-ch/installing-couchdb-on-raspbian-stretch-ccb2a7).

## Objective ##

This tutorial is a step by step explanation on how to install the [NOSQL database CouchDB](http://couchdb.apache.org/) on a Raspberry Pi with [Raspbian Stretch OS](https://www.raspberrypi.org/downloads/raspbian/).

## Plan ##

The plan for this tutorial is

- [Short introduction on what is CouchDB?](#01)
- Prerequisites for the Raspberry Pi.
- CouchDB download, build and installation.
- First run and check.
- Script for running CouchDB on boot.

---

# <span id="01">1. Short introduction on what is CouchDB?</span> #

CouchDB is an awesome and very reliable NOSQL database server which stores JSON documents. CouchDB can be run from a Raspberry Pi to big servers. A version built for mobile and desktop web-browsers is named [PouchDB](http://pouchdb.com/) and [Couchbase Lite](https://developer.couchbase.com/documentation/mobile/current/guides/couchbase-lite/index.html) is built for native iOS & Android apps. It is very easy to store and query documents with CouchDB databases and data can be replicated seamlessly with each other.

The [Apache CouchDB website](http://couchdb.apache.org/) says:

> Data Where You Need It
>
> Apache CouchDB™ lets you access your data where you need it by defining the **Couch Replication Protocol** that is implemented by a variety of projects and products that span every imaginable computing environment **from globally distributed server-clusters**, over **mobile phones** to **web browsers**. Software that is compatible with the Couch Replication Protocol include: PouchDB, Cloudant, and Couchbase Lite.

> Store your data **safely**, on your own servers, or with any leading cloud provider. Your web- and native applications love CouchDB, because it speaks **JSON natively** and supports binary for all your data storage needs. **The Couch Replication Protocol** lets your data flow seamlessly between server clusters to mobile phones and web browsers, enabling a compelling, **offline-first** user-experience while maintaining high performance and strong reliability. CouchDB comes with a **developer-friendly query language**, and optionally MapReduce for simple, efficient, and comprehensive data retrieval.

---

# 2. Prerequisite for the Raspberry Pi. #

This step by step tutorial has been tested many time with the following configuration:

**Hardware** : Raspberry Pi 2B, 3B and 3B+.

**OS**: Raspbian Stretch.

**Network**: you should have an Internet access to download and install software.

**SSH** : you can install CouchDB with SSH or in a Terminal session on the Pixels interface directly on the Raspberry Pi.

First, check that you have the correct OS version by executing in a Terminal:

	cat /etc/os-release

You should get something like:
	
	# PRETTY_NAME="Raspbian GNU/Linux 9 (stretch)"
	# NAME="Raspbian GNU/Linux"
	# VERSION_ID="9"
	# VERSION="9 (stretch)"
	# ID=raspbian
	# ID_LIKE=debian
	# HOME_URL="http://www.raspbian.org/"
	# SUPPORT_URL="http://www.raspbian.org/RaspbianForums"
	# BUG_REPORT_URL="http://www.raspbian.org/RaspbianBugs"

Then, ensure your system is up to date with:

	sudo apt-get update
	sudo apt-get dist-upgrade

---

# 3. Download, build and install software. #

Now, we add the Erlang Solutions repository and public key with:

	wget http://packages.erlang-solutions.com/debian/erlang_solutions.asc
	sudo apt-key add erlang_solutions.asc
	sudo apt-get update
	
	sudo apt-get --no-install-recommends -y install build-essential \
	pkg-config erlang libicu-dev \
	libmozjs185-dev libcurl4-openssl-dev
	
	sudo useradd -d /home/couchdb couchdb
	sudo mkdir /home/couchdb
	sudo chown couchdb:couchdb /home/couchdb
	
	cd
	
	wget http://mirror.ibcp.fr/pub/apache/couchdb/source/2.1.1/apache-couchdb-2.1.1.tar.gz   
	
	tar zxvf apache-couchdb-2.1.1.tar.gz
	cd apache-couchdb-2.1.1/
	
	./configure
	make release
	
	cd ./rel/couchdb/
	sudo cp -Rp * /home/couchdb
	sudo chown -R couchdb:couchdb /home/couchdb
	
	cd
	rm -R apache-couchdb-2.1.1/
	rm apache-couchdb-2.1.1.tar.gz
	rm erlang_solutions.asc

---

# 4. First run and check. #

For remote access:

	sudo nano /home/couchdb/etc/local.ini

=> change line

	#bind_address = 127.0.0.1

to:

	bind_address = 0.0.0.0

(! not prefixed by # !)

Now, you are ready to run CouchDB as couchdb user with:
	
	sudo -i -u couchdb /home/couchdb/bin/couchdb

---

# 5. Script for running CouchDB on boot. #

	mkdir /var/log/couchdb/
	sudo chown couchdb:couchdb /var/log/couchdb
	
Then, be sure that CouchDB is still running and in your Browser go into the Configuration page: [http://127.0.0.1:5984/_utils/#/_config](http://127.0.0.1:5984/_utils/#/_config). If you have defined an administrator you should be logged in as the administrator.

Click on the +Add Option and fill the form like here:

values are:

- `log` for Section,
- `file` for Name and 
- `/var/log/couchdb/couch.log` for Value.

Click on Create.

Now, create the service by editing a new file:

	sudo nano /lib/systemd/system/couchdb.service

and paste the following to the content of the editor:

	[Unit]
	Description=CouchDB Service
	After=network.target
	  
	[Service]
	Type=idle
	User=couchdb
	Restart=always
	ExecStart=/home/couchdb/bin/couchdb
	  
	[Install]
	WantedBy=default.target
	
Then, fix file permissions with:

	sudo chmod 644 /lib/systemd/system/couchdb.service

and instruct systemd to start the service during the boot sequence:

	sudo systemctl daemon-reload
	sudo systemctl enable couchdb.service

When you reboot the Pi the couchdb service should run:

	sudo reboot

and check service status using:

	sudo systemctl status couchdb.service

Also, either browse to page http://localhost:5984 to check that CouchDB server is up and running, or execute in your terminal:

	curl http://localhost:5984

which should reply something like :

	{"couchdb":"Welcome","version":"2.1.1","features":["scheduler"],"vendor":{"name":"The Apache Software Foundation"}}

Now, you are ready to enjoy all CouchDB server capabilities !

Have a look to CouchDB Documentation and enjoy PouchDB in your browser, mobile or NodeJS projects.

Your Feedback and suggestions for improvements would be welcome !  
joel