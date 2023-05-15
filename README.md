# vps-deployment-with-github-actions
vps deployment with github actions step by step guide in digital ocean as an exemple

##  - Create Node.js server

##  - Create React app

##  - Create GitHub repository

##  - Push code to GitHub
    here make sure to commit in the root not in the frontend folder
    and before that run this in your frontend folder 
    $ rm -rf .git

## - Create SSH Keys
    in .ssh folder in your computer run these: 
    1- $ $ cd .ssh 
    2- $ ssh-keygen -t rsa -b 4096 -C "youremail"
    give it a name for exemple "portfolio"

## - Create Digital Ocean linux server
    add ssh public key to your droplet 


## - Connect to Digital Ocean server with SSH
    $ ssh -i portfolio root@ipOfYourDroplet

## - update the server
    $ sudo apt update && sudo apt upgrade -y

## - Install Nodejs
    $ curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\sudo apt-get install -y nodejs

    Or: follow this link : https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04

## - Install and verify Nginx
    $ sudo apt-get install nginx -y && sudo systemctl enable nginx && sudo systemctl status nginx

    OR: follow this link : https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04

    ps: don't forget to update the server 

## - Install and configure mongoo
    $ wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add - && echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list && sudo apt-get update && sudo apt-get install -y mongodb-org && sudo systemctl start mongod && sudo systemctl status mongod && mongo --version 

    ## -configuration:
        $ sudo systemctl enable mongod.service && sudo chown -R mongodb:mongodb /var/lib/mongodb
        && sudo chown mongodb:mongodb /tmp/mongodb-27017.sock
        && sudo service mongod restart


## - Create a new user
    $ adduser imen (or any name you want it )
    then password , fullname ...

    // make this user as sudo:
        $ sudo usermod -aG sudo imen

    // add privilages to him:
        $

    login with your user:
    $ su - imen

    <!-- ps: to logout from any user just tap : $ exit -->

    now we need to enter to the directory whre we will create our project folder:
    $ cd /var/www/    <!-- ps: if you are in the root directory just tap $ cd ..  -->

    create the folder where you add your project
    $ sudo mkdir foldername  <!--foldername: for exemple portfolio -->

    $ cd foldername

## - Install GitHub actions runner
    before that lets prepare our ripo :
    in your repo -> settings -> actions -> runners -> new-self-hosted-runner
    you will get a list of command :
    1- # Download the latest runner package  // run it with sudo 
    2- # Extract the installer // run it with sudo
    after that run yhis :
    $ sudo chmod -R 777 /var/www/foldername  
        <!-- is used to change the permissions of the /var/www/myportfolio directory and all its files and subdirectories recursively -->

    3- # Create the runner and start the configuration experience // without sudo


## ## ## ## ## ## ## ## ## congrat now your github actions are ready to use ## ## ## ## ## ## ## ##

    run this : 
        $ sudo ./svc.sh install
    then :
        $ sudo ./svc.sh start
## - Configuring Nginx to display our project
    $ cd /etc/nginx/sites-available
    $ sudo nano default // so we can edit the configuration like this:
    <!-- 
            server_name your_domain.com;  # Replace 'your_domain.com' with your actual domain or server IP
            location / {
                proxy_pass http://localhost:3000;  # Proxy requests to the React app
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
            }
            root /var/www/foldername/_work/reponame/reponame
     -->

    ctrl x -> y -> ctrl c    // to save then exit

    $ sudo service nginx restart // to restart server nginx

## - Run Nginx without SUDO command 
    $ sudo visudo -f /etc/sudoers.d/imen
    add this on it
        imen ALL=(ALL) NOPASSWD: /usr/sbin/service nginx start, /usr/sbin/service nginx stop, /usr/sbin/service nginx restart
    ctrl x -> y -> ctrl c    // to save then exit

    now just reboot: 
    $ sudo reboot


## - Create GitHub actions workflows:
    in yout root folder create a folder -> .github
    inside of it create a folder -> workflows
    inside of it create a file called for exemple -> nodejs.js.yml
    inside of it add this code :
    <!-- 
    //////////////////////////////////////////
        name: Node.js CI

        on:
        push:
            branches: [main]
        pull_request:
            branches: [main]

        jobs:
        build:
            runs-on: self-hosted
            steps:
            - uses: actions/checkout@v2
            - name: Use Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
                node-version: ${{ matrix.node-version }}
            - run: |
                npm i --no-optional  # Add the --no-optional flag here
                cd frontend
                npm i --no-optional  # Add the --no-optional flag here
                npm run build
                cd ..
                pm2 stop 0
                pm2 start 0
                pm2 save
                sudo service nginx restart
    ////////////////////////////////////////////////////////////////////////
     -->

     ps: don't forget to use this on your express app in app.js
        app.use(express.static(path.join(__dirname, '/frontend/dist')));
     ps: if you use vite -> the build folder will be "dist" -> '/frontend/dist'
         id you use create-react-app -> the build folder will be "build" -> '/frontend/build'

## - Install Pm2
    in our project which is in :
    $ cd _work // you will get your repo
    $ cd reponame/reponame
    here will get all your project files includes your frontend folder with "dist" or "build" folder
    
    run this here :
    $ sudo npm install pm2@latest -g
    $ pm2 status
    now if you access to $ cd frontend // you will get a folder called "dist" or "build" 

## - Start React and Node on server
to start the server make sure you are in the root of your application where your frontend folder and backend files are located  then run 
    $ pm2 start --npm "foldername" --run start 
    // make sure you have in package.json of your express app in the script "start": "node app.js"

    then 
    $ pm2 save


ps: if you wanna access to any file to edit it you can run this command
    $ sudo nano nameOfFile.extension
