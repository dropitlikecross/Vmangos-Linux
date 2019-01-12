## **Compile Vmangos on Ubuntu 16.04**

**Steps:**


 1. Setup the Environment
 2. Download and Compile VMangos
 3. Import and update the database
 4. Configuration
 5. Extract the maps
 6. Run the server

## Install compilers and software environments

    sudo apt-get install build-essential gcc g++ automake git-core git autoconf make patch libmysql++-dev libtool libssl-dev grep binutils zlibc libc6 libbz2-dev cmake subversion libboost-all-dev wget tumx



## Download and compile ACE

    wget http://download.dre.vanderbilt.edu/previous_versions/ACE-6.5.0.tar.gz
    tar -zxvf ACE-6.5.0.tar.gz
    
After decompression, there will be an ACE folder and ACE_wrappers in the root directory.
Set the variables

    export ACE_ROOT=/root/ACE_wrappers 
    export LD_LIBRARY_PATH=$ACE_ROOT/lib:$LD_LIBRARY_PATH
Edit the settings file

    nano $ACE_ROOT/ace/config.h

Add:

    #include "ace/config-linux.h
Create a new file:

    $ACE_ROOT/include/makeinclude/platform_macros.GNU
Add:

    include $(ACE_ROOT)/include/makeinclude/platform_linux.GNU 
    INSTALL_PREFIX = /usr/local

Compile ACE:

    cd /root/ACE_wrappers 
    make && make install

## Download VMangos and the Database

Clone VMangos from the Git Repository

    git clone https://github.com/vmangos/core.git
    
Clone the Database from the Git Repository

    git clone https://github.com/brotalnia/database.git

## Compile Mangos

**Pre Compile Configuration**
TBB needs to be turned off before compiling 

    cd core/
    nano CMakeLists.txt

Make sure that the TBB field is set to 1 like in the picture below.
![enter image description here](https://fx.fklds.com/wp-content/uploads/2018/12/1aaf76c402f520.jpg)

**Compile**
Note: You can replace *opt/lh* with wherever your want the binary files to be created however you need to change the rest of the locations in the guide to match.

    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX=/opt/lh ../
After this completes run:
   
     make && make install

## Import and update the database

Go into MYSQL and follow the steps in sequence.

    mysql -u root -p
    CREATE DATABASE IF NOT EXISTS realmd DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
    CREATE DATABASE IF NOT EXISTS characters DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
    CREATE DATABASE IF NOT EXISTS mangos DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
    CREATE DATABASE IF NOT EXISTS logs DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
    
Create a new user and authorize them. **I strongly recommend changing the name from 'mangos'.**

    create user mangos identified by 'mangos';
    grant all on realmd.* to mangos@'localhost' identified by 'mangos' with grant option; 
    grant all on characters.* to mangos@'localhost' identified by 'mangos' with grant option; 
    grant all on mangos.* to mangos@'localhost' identified by 'mangos' with grant option; 
    grant all on logs.* to mangos@'localhost' identified by 'mangos' with grant option;

**SQL Migration**

Pay attention to the version of the database. It **is very important to** check the first date of the database patch **! ! If the database version is wrong, you will experience problems. You have to start from the earliest date**

    cd /root/core/sql/migrations
    ls

![enter image description here](https://fx.fklds.com/wp-content/uploads/2018/12/1a3bfe50621c5c.jpg)
In this example June 8th is the earliest version so we begin from there.

**Uncompress the file**

    cd database/
    tar -zxvf world_full_08_june_2018.7z

**Import the data**

    cd /root/core/sql
    mysql -u mangos -p realmd < logon.sql
    mysql -u mangos -p logs < logs.sql
    mysql -u mangos -p characters < characters.sql
    mysql -u mangos -p mangos < /root/database/world_full_08_june_2018.sql


**Update the base database files:**

    cd core/sql/migrations/
    chmod +x merge.sh
    ./merge.sh
    mysql -u mangos -p realmd < logon_db_updates.sql
    mysql -u mangos -p logs < logs_db_updates.sql
    mysql -u mangos -p mangos < world_db_updates.sql

## Configuration

Create the data and log folders

    cd /opt/lh
    mkdir data
    mkdir log

Edit Mangosd.conf & Realmd.conf
*Note in some cases they are named mangosd.conf.dist & realmd.conf.dist you will have to rename them.*

    Nano etc/mangosd.conf
![
](https://fx.fklds.com/wp-content/uploads/2018/12/13ac569b05ff2e.jpg)

    nano etc/realmd.conf
![
](https://fx.fklds.com/wp-content/uploads/2018/12/18de26ff0a25bc.jpg)


**Configure the authentication server**

You need to change the following values
**Servername** , **Server IP Address** and if you wish, change the port from 8085 but make sure to update the .conf files to match

    mysql -u root -p
    
    use classicrealmd;
    DELETE FROM realmlist WHERE id=1;
    INSERT INTO realmlist (id, name, address, port, icon, realmflags, timezone, allowedSecurityLevel)
    VALUES ('1', 'Servername', 'Server IP Address', '8085', '1', '0', '1', '0');
    exit;

In your local WOW client folder, find the realmlist.wtf file and edit it with Notepad. The IP and PORT here are the same as the database.

    realmlist.wtfset realmlist 152.178.2.154

## Extracting Maps

Use google and find a guide.

## Running the Server

There are many ways to do this but we'll use Tmux so that you can watch what's happening on screen.

    tmux
    cd /opt/lh/bin
    ./mangosd -c ../etc/mangosd.conf

Separate tmux, first press ctrl + b then press shift + 5

Press ctrl + b then press o to switch to the other half of the screen

    cd /opt/fklserver/bin
    ./realmd -c ../etc/realmd.conf

## Add an account

In the mangosd window you can enter the following commands.

account create [username] [password]    //Create user
account set addon [username] [0 to 3]   //Set the WOW environment, default is 0
account set gmlevel [username] [0 to 3]  //User level 0 = player 1 and 2 = GM 3 = administrator
server shutdown [delay]                  //how many seconds to shut down the server
