# Monitor your FRITZ!Box Router with Grafana
FRITZ!Box from AVM has a very limited monitoring functionality on board. There is no SNMP available on FFRITZ!Box and there is no chance you could get a graph of your traffic statistics from it.
With Collectd, InfluxDB and Grafana its possible to get rid of this limitation. Grafana has even a ready to use Dashboard for monitoring FRITZ!Box:
![Grafana](https://grafana.com/api/dashboards/713/images/2258/image)

## Installation
```
sudo apt-get update && sudo apt-get -y upgrade
# To collect the data out of FRITZ!Box the python script "fritzcollectd" comes in play. 
# So let's first install python packet manager python-pip
sudo apt-get install -y python-pip
sudo apt-get install -y libxml2-dev libxslt1-dev
# Installing fritzcollectd and its prerequisites would take about 30 minutes
sudo pip install fritzcollectd
```
To be able to collect the monitoring data from FRITZ!Box UPnP must be activated:
>Heimnetz :: Netzwerk :: Netzwerkeinstellungen

![UPnP](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/85863940829962f886f51bb191d73615.png)

Create a user account on FRITZ!Box for queries we're going to make on the router

![User Creation](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/9c55f8ea21d20fa03570d6ebc39db0e2.png)

## Data Collection
In this section we're going to configure collectd to collect data from FRITZ!Box every 10 seconds.

###### Install collectd:
```
sudo apt-get install -y collectd
```
###### Configure collectd:
```
sudo nano /etc/collectd/collectd.conf
```
###### Uncomment the following lines:
 - #LoadPlugin python
 - #LoadPlugin network
###### Network plugin section to write data in InfluxDB:
```
<Plugin network>
    Server "127.0.0.1" "25826"
</Plugin>
```

###### Add the following block at the end of config file for python plugin (replace username,password and address):
```
<Plugin python>
    Import "fritzcollectd"

    <Module fritzcollectd>
        Address "192.168.1.1"
        Port 49000
        User "username"
        Password "password"
        Hostname "FritzBox"
        Instance "1"
        Verbose "False"
    </Module>
</Plugin>
```

## Backend
We'll be using InfluxDB as database for saving the data.
###### Install InfluxDB and enable it to start automatically
```
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/debian stretch stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt-get update
sudo apt install influxdb
sudo systemctl enable influxdb
sudo systemctl start influxdb
```

###### Configure InfluxDB:
```
sudo nano /etc/influxdb/influxdb.conf
```
###### Find the block [[collectd]] and replace it with the following:
```
[[collectd]]
  enabled = true
  bind-address = "127.0.0.1:25826"
  database = "collectd"
  typesdb = "/usr/share/collectd/types.db"
```
###### Restart collectd and InfluxDB to activate the new config
```
sudo systemctl restart collectd 
sudo systemctl restart influxdb
```

## Frontend
The last packet to install is Grafana.
###### Download the packet manually and install it
```
mkdir -p tmp
cd tmp && wget https://dl.grafana.com/oss/release/grafana_6.0.1_armhf.deb
sudo dpkg -i grafana_6.0.1_armhf.deb
```

###### Enable auto start for Grafana
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server
sudo /bin/systemctl start grafana-server
```
## Configuring Grafana and the connections
Open the console of Grafana under http://raspberrypi-ip:3000/login. The default username and password is admin:admin. You'll be asked to change it on the first login:
![Grafana Console](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/b4806c57c622543298ee39520bb687b5.png)

![Grafana Console](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/b78cabf6e6ebcacf92b4cade46ee9a4e.png)

###### Add Data Source
Over navigation menu or by visiting the url http://raspberrypi-ip:3000/datasources add a new data source according the screenshot:
![Grafana Console](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/b02f416e4d98890e004bc4ba0af91f1c.png)

After clicking on "Save and Test" if the data source is functional it'll be added.

## Add Dashboard
###### Hover your mouse on the "+" in the navigation menu and click on import. 
![Dashboard Import](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/d7ChmhUybQPnznal.png)

###### In the first empty field type 713 (ID for FRITZ!Box Dashboard on Grafana's repository) and on the next step select influxdb_collectd as data source
![Dashboard Import](https://blog.butenostfreesen.de/2018/10/11/Fritz-Box-Monitoring-mit-Grafana-und-Raspberry/SrhYQOWcLORLKO4F.png)
