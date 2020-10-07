##### WORK IN PROGRESS ... #####

3. :blue:`(Optional / You can proceed without these steps if you just want to use the VSS structure as is.)` You can extend or modify the existing VSS data structure during runtime by using `kuksa.val/vss-testclient/testclient.py`. The followings describe from installing python dependencies, running `testclient.py` to extending or modifying the VSS structure.

3-1. Install requirements (Python 3.8)::

    $ sudo add-apt-repository ppa:deadsnakes/ppa
    $ sudo apt update
    $ sudo apt install python3.8

3-2. Install requirements (websockets, cmd2, pygments)::

    $ pip3 install websockets cmd2 pygments

3-3. Make sure that the kuksa.val server is already up and running, then navigate to the directory, `kuksa.val/vss-testclient/`, and run `testclient.py` with Python 3.8::

    $ python3.8 testclient.py

3-4. If connected to the server successfully, you would be in the VSS Client shell. To get an admin access to the server, you need to assign a JSON token. Command the following::

    VSS Clinet> authorize ../certificates/jwt/super-admin.json.token

3-5. 
authorize ../
getMetaData Vehicle.Speed
setValue Vehicle.Speed 200
setValue Vehicle.Private.ThurstersActive true

shall cat modified.json
updateVSSTree ../build/src/modified.json

getMetaData Vehicle.Speed
getValue Vehicle.Speed
setValue Vehicle.Private.ThurstersActive true
getValue Vehicle.Private.ThurstersActive