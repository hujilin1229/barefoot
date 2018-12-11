# Barefoot for Harbin

## Map server

1. Install prerequisites.

    - Docker Engine (version 1.6 or higher, see [https://docs.docker.com/installation/ubuntulinux/](https://docs.docker.com/installation/ubuntulinux/))

2. Download the map data and extract

    ``` bash
    curl http://download.geofabrik.de/asia/china-latest.osm.pbf -o barefoot/map/osm/china.osm.pbf
    cd barefoot/map/osm/
    osmosis --read-pbf file=china-latest.osm.pbf --bounding-box left=126.506130 right=126.771862 bottom=45.657920 top=45.830905 --write-pbf file=harbin.osm.pbf
    ```

3. Build Docker image.

    ``` bash
    cd barefoot
    docker build -t barefoot-map ./map
    ```

4. Create Docker container.

    ``` bash
    docker run -it -p 5432:5432 --name="barefoot-harbin" -v ${PWD}/map/:/mnt/map barefoot-map
    ```

    To allow remote access to the database
    ```bash
    sudo vim /etc/postgresql/9.3/main/pg_hba.conf
    # add the line in the end
    host all all all md5
    # restart
    sudo service postgresql restart
    # save the container status
    $ docker commit `container id` barefoot-harbin
    ```

    Once we have created the docker container, we can start it with the following command in the future
    ```bash
    docker start --interactive barefoot-harbin
    ```

5. Import OSM extract (in the container).

    ``` bash
    root@acef54deeedb# bash /mnt/map/osm/import.sh
    ```

    _Note: To detach the interactive shell from a running container without stopping it, use the escape sequence Ctrl-p + Ctrl-q._

6. Make sure the container is running ("up").

    ``` bash
    docker ps -a
    ...
    ```

    _Note: The output of the last command should show status 'Up x seconds'._

##### Matcher server

_Note: The following example is a quick start setup. For further details, see the [wiki](https://github.com/bmwcarit/barefoot/wiki#matcher-server)._

1. Install prerequisites.

    - Maven (e.g. with `sudo apt-get install maven`)
    - Java JDK (Java version 7 or higher, e.g. with `sudo apt-get install openjdk-1.7-jdk`)

2. Package Barefoot JAR. (Includes dependencies and executable main class.)

    ``` bash
    mvn package
    ```

    _Note: Add `-DskipTests` to skip tests._

3. Start server with standard configuration for map server and map matching, and option for GeoJSON output format.

    ``` bash
    java -jar target/barefoot-<VERSION>-matcher-jar-with-dependencies.jar --geojson config/server.properties config/oberbayern.properties
    ```

    _Note: Stop server with Ctrl-c._

    _Note: In case of 'parse errors', use the following Java options: `-Duser.language=en -Duser.country=US`_

4. Test setup with provided sample data.

    ``` bash
    python util/submit/batch.py --host localhost --port 1234  --file src/test/resources/com/bmwcarit/barefoot/matcher/x0001-015.json
    SUCCESS
    ...
    ```

    _Note: On success, i.e. result code is SUCCESS, the output can be visualized with [http://geojson.io/](http://geojson.io/) and should show the same path as in the figure above. Otherwise, result code is either TIMEOUT or ERROR._
