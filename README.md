# EmComMap
## Live multi-user annotatble map for emergency communications

### Containerized by Jay Land (KF7ITF), v.jay.land@gmail.com
### Note there has been no code changes from [DanRuderman/EmComMap](https://github.com/DanRuderman/EmComMap). This is just a containerization with updated Apache2, Couchdb, and the Tileserver.
<img src="Resources/map_image.png" width="450">

### Home page
[EmComMap](http://emcommap.org)

### Demo
[Demo server](http://app.emcommap.org/EmComMap/html/index.html)

### Features
* Interactive map with symbols for operators, locations, and incidents
* Incidents defined by geographic region and time
* Text-based message traffic with associated precedence (e.g. priority, emergency)
* Attach files to messages
* Operator status reports
* Location status reports
* Tactical call signs
* Upload locations by latitude/longitude
* Download message logs
* Message sorting and location filtering
* Persistent storage on server database, with replication

### Deployment
* Docker and Docker-compose must be installed for this version to work.
* For the initial run your computer must have access to the internet. After that no internet connection is needed to build the containers.
* Can be encrypted (https) or unencrypted (http, suitable for Amateur Radio)
* MESH networking
* Internet / intranet
* Relies on three server platforms, deployed in a container stack:
  1. CouchDB (DB)
  2. Map tile server (Map)
  3. Apache2 (Web)
* The following steps will have the stack up and running.
  1. Unzip the EmComMap.zip into a location accessable to your user.
  2. Open a command promot and navigate the the EmComMap folder location unziped in step 1.
  3. Enter the command 'docker-compose up -d'
* Access the couchdb management web page:
  1. `http://localhosts:5984/_utils`
  2. User name is: `admin`
  3. Passworkd is: `password`
* Access the tileserver management web page:
  1. `http://localhosts:8888`
* Access the EmComMap login web page:
  1. `http://localhosts:8080`
  2. Usernames and passwords:
    a. Admin user:`admin` Password: `password`
    b. EmComMap user: `user1` Password: `password`
* Important folder locations and descriptions:
  1. Apache2
    a. `.\EmComMap\web\data` - EmComMap config and source files
    b. `.\EmComMap\web\config` - Apache2 config files httpd.conf.
  2. Couchdb
    a. `.\EmComMap\couchdb\data` - Database files
    b. `.\EmComMap\couchdb\data\etc` - Couchdb config files
  3. Tileserver-gl
    a. `.\EmComMap\tileserver-gl\data` - Map tile files

### User manual
* See [EmComMap User Guide](html/Documentation/EmComMap_user_guide.pdf)


#### CouchDB database (http://couchdb.apache.org/)

Docker-compose was use to create pull the official image for Docker Hub. The below is the section used to setup the image.

```
db:
    image: couchdb
    ports:
      - "5984:5984" #Maps the couchdb web connection to http://localhost:5984 to get to the couchdb management interface you would go to http://localhost:5984/_utils
    volumes:
      - .\couchdb\data:/opt/couchdb/data #Makes a persistant mount for the database files
      - .\couchdb\data\etc:/opt/couchdb/etc #Makes a persistant mount for the couchdb files
    restart: always
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=password
```

#### The next section is just for reference as the changes have been made to the files in the zip.
Edit the file `.\EmComMap\couchdb\data\etc\local.ini`, for example using  


1. Insert the line:
```enable_cors = true```
beneath the line that reads
```[httpd]```

1. Add the following lines to the end of the file:
```
[cors]
	origins = *
	credentials = true
	methods = GET, PUT, POST, HEAD, DELETE
	headers = accept, authorization, content-type, origin, referer, x-csrf-token
```

### Adding Users

#### You will however, have to acces the configuration tool to add users to the system using the admin user (admin:password). One user is currently setup (user1:password).

Set up databases using web configuration tool. Direct your browser to the couchdb container/service must be running to set this up.
`http://localhosts:5984/_utils/`, 

1. Set an admin password. A link in the lower right-hand corner of the web page should give you the option of setting an administrator password.

1. Verify the installation. There is a Verify link (Check mark Icon) to do this on the left hand side of the page.

1. Create databases for EmComMap.
   * Click *Databases*, which is on the left hand side
   * On the upper right, click *Create Database*
   * Create a database called *emcommap*
   * Using the same method, create a second database called *emcommap_attachments*
  
To create users:
1. Click *Databases*, which is on the left hand side
1. In the main pane, you will see a link to the database *_users*. Click on it.
1. Click *New Document* on the upper right, which will bring up a document containing one line
1. Click the *Source* tab on the upper right
1. Double-click on the document to get into edit mode, and replace the document's text with the following:

   ```
   {  
   "_id": "org.couchdb.user:username",  
   "name": "name_of_user",  
   "type": "user",  
   "roles": [],  
   "password": "plaintext_password"  
   }
   ```
   
1. In the text, replace *username* with the desired user name, *name_of_user* with the user's name, and *plaintext_password* with the user's password (it will be stored as a hash, not in plain text).
1. Click *Save Document* in the upper left corner

You must give each user that you create permission to access the EmComMap databases, as follows:
1. Go to *Databases* (menu on the left).
1. Click on the database *emcommap*
1. Click on *Permission* in the main pane.
1. You will add to the *Members* area of the dialog. In the *Names* field you will fill in each user.
1. Click *Add User* to complete.
1. Repeat these same steps for the database *emcommap_attachments*

#### Apache2 web server (https://httpd.apache.org/)

Note that while Apache2 is used in this example, other web servers should perform just as well.

Docker-compose was use to create pull the official image for Docker Hub. The below is the section used to setup the image.

```
web:
    image: httpd:2.4
    ports:
      - "8080:80" #Maps the EmComMap web page to http://localhost:8080
    volumes:
      - .\web\data:/usr/local/apache2/htdocs #Makes a persistant mount for the EmComMap files
      - .\web\config:/usr/local/apache2/conf #Makes a persistant mount for the Apache2 config files httpd.conf. The addition is to further point to the html folder where the EmComMap files are.
    depends_on: #Ensures that the Apache2 server will not come up until the couchdb and tile servers do.
      - db
      - map
    links: #Allows them to see each other on the stack's network.
      - "db"
      - "map"
    restart: always
```

#### The EmComMap source files are located in the .\EmComMap\web\data folder. The Apache2 config files (httpd.conf) is located in the .\EmComMap\web\config folder.

#### Map tile server (https://github.com/maptiler/tileserver-gl)

Docker-compose was use to pull the image for Docker Hub. The origional called for image had a cruppted manifest file and could not be pulled down. The below is the section used to setup the image.

```
map:
    image: maptiler/tileserver-gl
    ports:
      - "8888:80" #Maps the tileserver-gl web management to http://localhost:8888
    volumes:
      - .\tileserver-gl\data:/data #Makes a presistant mount for the map tiles
    restart: always 
```

Download map tiles from [OpenMapTiles](https://openmaptiles.com/downloads/planet/). You can download the entire planet or just a region of interest by navigating to that location. For this exmple we will download the Los Angeles region. Whichever geographic area you choose, you will want to download the *OpenSteetMap* tiles. Note you will need to create an account to download tiles. OpenMapTiles will provide you a *wget* command to download or a direct download. The tile files should be placed in the `.\EmComMap\tileserver-gl\data folder`.

To test that the tile server is functional, enter the following URL into your browser:  

```http://localhost:8888``` It should bring up a single tile of the world.


## Configuration

#### EmComMap configuration

NOTE: If you are using the EmComMap.zip all of the following is configured with the execption of the e-mail address.

To configure your installation, edit the file `.\EmComMap\web\data\html\config.js`. The folder is set up as a presistant mount for the web container and mounted to the correct location in the container (See docker-config.yml for services>web>volumes). Toward the top of the file you will see these lines:

```
//const RUN_LOCATION = "local"; 
const RUN_LOCATION = "my-install";

if(RUN_LOCATION == "my-install") {
    //var TILE_SERVER = 'http://<host>:8080/styles/klokantech-basic/{z}/{x}/{y}.png'; // Original but due to the tileserver upgrade the line below is needed. 
    var TILE_SERVER = 'http://localhost:8888/styles/basic-preview/{z}/{x}/{y}.png'; // Changed due to having to use a different tileserver-gl than above. This line was found by going to the tile server and clicking on the xyz link and coping the infomation here.
    var TILE_SERVER_OPTS = {
	maxZoom: 18,
	attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
	    '<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
	    'Server courtesy of <a href="https://openmaptiles.com/">OpenMapTiles</a>'
    };
    var DEFAULT_DB_HOST = '<host>';
```

Within them, change the following:
1. Change the RUN_LOCATION string to *my-install* instead of *local*
1. Change both instances of `<host>` to the name or IP address of your EmComMap server. Note that if you have the CouchDB server on a different computer, then use that computer's address for *DEFAULT_DB_HOST*.

Set the value of `ADMIN_EMAIL` to be the email address for credentials requests.

Additionally, if your deployment is for testing only set the value of `TEST_MODE` to `true` in `config.js`. This will put the text TESTING in bold red font at the top of the application and precede all messages with "TESTING:". The purpose is to ensure that test traffic is not mistaken for a real-world emergency.

#### Database replication
Although I have not attempted it, it should be straightforward to replicate the CouchDB databases across servers, thus providing a route to failover. Please consult the CouchDB documentation on how to achieve this.

### Note on encryption and Amateur Radio use
Amateur radio based applications (e.g. supporting MESH networking) may not encrypt data transmissions.

* [CouchDB does not by default enable encryption](https://stackoverflow.com/questions/44585302/what-encryption-mechanism-is-used-in-couchdb), as it uses the http rather than the https protocol. The relevant lines in the configuration files (*/etc/couchdb/default.ini* and */etc/couchdb/local.ini*) contain the text `httpsd`, and are by default commented out. The default port for CouchDB's https encrypted communications is 6984, so if you run the command `$ netstat -plnt  | grep 6984`, you should see no output unless CouchDB is listening on an https socket. If you wish to enable SSL based encryption, see [this article](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=48203146).

* Apache2 does not by default enable encryption over SSL. To check that the SSL port (443) is not open, run the command `$ netstat -plnt  | grep apache2`, which should only show a line for port 80 and not for port 443. If you wish to disable SSL, comment out any lines containing `Listen 443` in the file */etc/apache2/ports.conf*.

* Tileserver-gl as installed above does not use encryption.

### Thanks to those who kindly provided these software libraries leveraged by EmComMap:
* [Leaflet](https://leafletjs.com/)
* [FileSaver.js](https://github.com/eligrey/FileSaver.js/)
* [filesizejs](https://filesizejs.com/)
* [jquery-timepicker-ui-addon](https://www.npmjs.com/package/jquery-ui-timepicker-addon)
* [tablesorter](https://github.com/Mottie/tablesorter)
* [js-cookie](https://github.com/js-cookie/js-cookie)
* [moment.js](https://momentjs.com/)
* [PouchDB](https://pouchdb.com/)
* [smart-time-ago](https://github.com/pragmaticly/smart-time-ago)
* [xlsx-populate](https://github.com/dtjohnson/xlsx-populate/tree/master/browser)

### Dan Ruderman (K6OAT) emcommap@gmail.com - EmComMap question
### Jay Land (KF7ITF) v.jay.land@gmail.com - Container question
