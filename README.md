# Before you start

You will need to find and download an unpacked LEGO Universe client. Make sure the client you get is unpacked and that its hash matches the ones mentioned in the [repo README](https://github.com/DarkflameUniverse/DarkflameServer#resources)

Using a packed client is possible, however it will need to be unpacked before it can be used. Instructions on how to do that can be also be found in the repo README.

**This tutorial was done in Windows 11. The way some things are accessed might be different in older versions of Windows**

# Setup

### Programs to install
You will need to install the following software

 - [7zip](https://www.7-zip.org/)
 - [Visual Studio C++](https://visualstudio.microsoft.com/vs/features/cplusplus/)
 - [CMake](https://cmake.org/download/)
 - [git](https://git-scm.com/download/win)
 - [mariadb](https://mariadb.com/downloads/community/community-server/)
	 > Set the root password as root
 - [sqlite browser](https://sqlitebrowser.org/dl/)

>Make sure, when asked, to add the program to the system PATH

### Cloning the server code repository

 1. Navigate to your `Documents` folder 
 2. Right click in this folder and select `Show More Options > Git Bash Here`
	 > If you are on Windows 10, you can skip the `Show More Options` option
 3. In the terminal window that popped up run `git clone --recursive https://github.com/DarkflameUniverse/DarkflameServer`

<br>

### Building the server

 1. Navigate to the `DarkflameServer`  folder
 2. Right click in the folder and select `Open in Windows Terminal`
	 > If you are on Windows 10 it might be called Powershell
3. Run `./build.sh`
	> If the build succeeded and the following files `MasterServer.exe`, `WorldServer.exe`, `AuthServer.exe`, `ChatServer.exe` are **not** in the build folder, you will need to move them from `build/Release` into `build`

While the server is building we will prepare the client files

<br>

### Preparing client files

#### Extracting the client
 1. Navigate to where you downloaded the client and move it to your `Documents` folder
 2. Right click the .rar client file and select `7-Zip > Extract files...` Keep the default settings (this will take some time)
	 >If your client is already extracted, move the folder to `Documents` 
3. You should now have a folder called something like `LEGO Universe (unpacked)`
4. You can now delete the .rar file if you want

#### Setting up resource directory

1. Go to the `res` folder of your client `LEGO Universe (unpacked)/res` and copy the following files and folders to the `res` folder in `Documents/DarkflameServer/build/res`   ||  
			 `macros`, 
			 `BrickModels`, 
			 `chatplus_en_us.txt`, 
			 `names`, and `maps`
2. Copy the `navmeshes.zip` file from `Documents/DarkflameServer/build/resources` and paste it in `Documents/DarkflameServer/build/res/maps/`
3. Unzip `navmeshes.zip` (right click > Extract all) into the maps folder

#### Setting up locale directory 

 1. Navigate to the client `locale` directory 
 2. Copy the `locale.xml` file the the server `build/locale` directory

#### Client Database
1. Go back to the client `res` folder of the client and find the `cdclient.fdb` file
2. Go to https://fdb.lu-dev.net/ (this site will convert the client database to one there server can use)
3. Make sure `FDB to SQLite` option is selected
4. Upload the `cdclient.fdb` file
5. When it is done, move the downloaded file `cdclient.sqlite` to the server `res` folder
6. Rename the file to `CDServer.sqlite` 

#### Check
The build folder in the server should now look like this:
-   AuthServer.exe
-   ChatServer.exe
-   MasterServer.exe
-   WorldServer.exe
-   authconfig.ini
-   chatconfig.ini
-   masterconfig.ini
-   worldconfig.ini
-   **locale/**
    -   locale.xml
-   **res/**
    -   CDServer.sqlite
    -   chatplus_en_us.txt
    -   **macros/**
        -   ...
    -   **BrickModels/**
        -   ...
    -   **maps/**
        -   **navmeshes/**
            -   ...
        -   ...
-   ...

### Server mySQL database
1. In the Start Menu start typing `mariadb` and open the program that is called `Command Prompt (MariaDB XXX (x64))` (where XXX is the version number)
2. In the terminal that just came up, log into the root user by running the following `mysql -u root -p`
	> the password when prompted will be `root`
3. Once logged in, run `CREATE OR REPLACE USER 'darkflame'@'localhost' IDENTIFIED BY 'password';`
	>  `darkflame` is the database username. Replace `password` with a password of your choice. This will be the database password
4. Then run `CREATE OR REPLACE DATABASE darkflame;`
	> `darkflame` is again the name of the database
5. Then run `GRANT ALL PRIVILEGES ON darkflame.* TO 'darkflame'@'localhost';`
6. Then run `FLUSH PRIVILEGES;`
7. Then exit by running `EXIT;`
8. You can now close the terminal
		
 # Server Configuration

### .ini files
In the `authconfig.ini`, `chatconfig.ini`, `masterconfig.ini` and `worldconfig.ini` files, add the database IP (`localhost`) to `mysql_host=`, the database name (`darkflame`) to `mysql_database=`, database username (`darkflame`) to `mysql_username=`, and the database password to `mysql_password=`

The end result should look like this

    # MySQL connection info:
    mysql_host=localhost
    mysql_database=darkflame
    mysql_username=darkflame
    mysql_password=XXXYYYZZZ
>Where `XXXYYYZZZ` is the password you chose earlier 

Next, in the `authconfig.ini`, `chatconfig.ini` and `masterconfig.ini` files, add the server IP (`localhost`) to `external_ip=`

Should look like 

    external_ip=localhost


### Database migrations

#### mysql

 1. Navigate to the `build` folder of the server and launch a terminal
 2. Run `./MasterServer.exe -m`
	 > If you get a message along the lines of `Finished running migrations`, you are good to go
 4. You can now close the terminal window

#### sqlite

 1. Open `DB Browser`
 2. Click on `Open Database` and select the `CDServer.sqlite` file located in the server `res` folder
 3. Switch to the `Execute SQL` tab
 4. Click on the open file button
 ![enter image description here](https://i.imgur.com/fDYTc1m.png)
 5. Navigate to the `migrations/cdserver` folder and select all the files and then hit `Open`
	 >If you are running a migration that has been recently added, only select the migration  file you want to run
 6. It should now look something like this (all the ones you selected should appear in separate tabs)
 ![enter image description here](https://i.imgur.com/sMB1wHH.png)

7. Switch to the tab with the lowest number, click the run button and then go to the next tab and repeat
![enter image description here](https://i.imgur.com/1hUoptA.png)
8. After all tabs have been ran, click the `Write Changes` button on the top
9. You can now close the window

#### Setup Admin Account

 1. Open a terminal in the `build` folder of the server and run `./MasterServer.exe -a`
 2. Enter a username
 3. Enter a password
 4. These credentials are what you are going to use to login to the game with **REMEMBER THEM**

 

# Client Configuration  

### boot.cfg

 1. Open you client folder and open the boot.cfg file in a text editor
 2. If it isn't already, set `AUTHSERVERIP=0:` to `localhost`

 Should look like this

    AUTHSERVERIP=0:localhost,

3. Save and close the file


### AG Survival Fix
The client has a typo in one of the scripts, and will cause the player to not load correctly in one of the minigames. To fix it, follow the steps below

1. In client, navigate to `res/scripts/ai/minigame/survival`
2. Right click the `l_zone_survival_client.lua` file and select `Properties`
3. In the properties, uncheck the `Read-only` check box at the bottom if it is checked
4. Hit `Apply` then `OK`
5. Open `l_zone_survival_client.lua` in a text editor and go to line 617
	> If you are using Notepad, hit `Ctrl+G` and type in 617 to go to line 617
6. Change  `PlayerReady(self)`  to  `onPlayerReady(self)`

Should look like the following when done

    function onTimerDone(self, msg)
		if msg.name == "Try_Freeze_Again" then    
	        freezePlayer(self, true)
	    elseif msg.name == "Try_Ready_Again" then
	        onPlayerReady(self)
	    end
	end


8. Save and close the file


# Running The Server

### Starting the server

 1. Open a terminal in the build folder of the server and run `./MasterServer.exe`
 2. A bunch of terminal windows should popup, this is normal
	 > If you get a bunch of firewall requests, close them out. You only need to accept them if you want make your server accessible outside your network (not covered in this tutorial)
 3. Start your client and login

### Shutting down the server

 1. Go to the main terminal you opened and hit `Ctrl+C`
 2. After a few seconds all the terminal windows (except the main one) should close
 3. Server is now shutdown
 4. Close the main terminal

# Updating The Server
As of the writing of this guide the DLU codebase is still actively being contributed to and as such, the code base is constantly being updated. Some of these updates include bug fixes or feature additions. It is recommend to update the server at least once every 2 weeks.

#### Update steps
Open a terminal in the server folder and run the following

 1. `git pull --recurse-submodules`
 2. `./build.sh`
	 > If the build succeeded and the following files `MasterServer.exe`, `WorldServer.exe`, `AuthServer.exe`, `ChatServer.exe` are **not** in the build folder, you will need to move them from `build/Release` into `build`

	> Running the build script will automatically run mysql migrations
3. Run any new sqlite migrations. See the `Server Configuration > Database Migrations > sqlite` for how to run sqlite migrations 
