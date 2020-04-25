# turkopticon

This is the Rails application for Turkopticon, an employer review system for Amazon Mechanical Turk.



## Installation

----

Since the application is relatively old, we will be using versions of software that play well together. You will find a lot of the instructions are specific to MacOS since that is the device I am working with — if you face an unexpected error, please create an [`issue`](https://guides.github.com/features/issues/) and include information about your device [here](https://github.com/fiveplusone/turkopticon-rails-app/issues).

0. ### Get the code

The commands in the boxes need to be entered into the terminal. If a line in the box starts with a `#`, it's intended to be a comment, and should be reviewed. You can learn more about commands, and the terminal [here](https://towardsdatascience.com/a-quick-guide-to-using-command-line-terminal-96815b97b955). 

You must have git already installed to run the first command. If you do not, you can install it by following [this guide](https://www.atlassian.com/git/tutorials/install-git).


```bash
git clone https://github.com/fiveplusone/turkopticon-rails-app.git
```

```bash
cd turkopticon-rails-app
```



1. ### Ruby 1.8.7

Before you run the following two commands, you will need to ensure that gpg has been installed on your computer. This is perhaps a good time to familiarize yourself with package mangers if you are not already. On Windows systems you may have good luck with [chocolatey](https://chocolatey.org/). For MacOS users, I **strongly** recommend installing [brew](https://brew.sh/). If you are a linux user, you will use either apt-get or yum based on your system. These package managers will make the following steps much easier.

Once you have a package manager ready, you should install gpg. For MacOS users that command will be as follows:

```bash
# note this is only for macos and you must install brew first
brew install gpg
```



##### Prepping RVM

```bash
# make sure gpg is installed
echo 409B6B1796C275462A1703113804BB82D39DC0E3:6: | gpg --import-ownertrust
```

```bash
echo 7D2BAF1CF37B13E2069D6956105BD0E739499BDB:6: | gpg2 --import-ownertrust
```



We will then install [rvm](https://rvm.io/) which will help us manage our older install of ruby.

```bash
\curl -sSL https://get.rvm.io | bash
```



##### Prepping OpenSSL version 1.0.x

We are **almost** ready to install ruby. However, since we are interested in an older version of ruby, we cannot directly install it since it is [not compatible with versions 1.1+ of openssl](https://github.com/rvm/rvm/issues/4781). So you will need to install openssl version 1.0.x. 

On a mac, if you have [`brew`](https://brew.sh/), you can run the following command:

```bash
brew install rbenv/tap/openssl@1.0
```

On MacOS, your directory will be `/usr/local/opt/openssl@1.0`. 

Once you have OpenSSL version 1.0.x installed, export its base directoy into your environment as a variable so we can use it when installing ruby. Exporting is done using the following command:

```bash
# the following command assumes you installed on macos using brew
# the directory at which you've installed openssl may be different
export OPENSSL_1_0_DIR="/usr/local/opt/openssl@1.0"
```



##### Install Ruby (finally)

```bash
# the following command will correctly install ruby
rvm install ruby-1.8.7-p374 --with-openssl-lib=${OPENSSL_1_0_DIR}/lib --with-openssl-include=${OPENSSL_1_0_DIR}/include --with-openssl-dir=${OPENSSL_1_0_DIR}
```

```bash
# this command will tell the system to use our version of ruby
rvm use ruby-1.8.7-p374
```



##### Test Installation

To test ruby has been correctly installed, try installing `rails` with the following command. Rails is an integral part of the application, and we need to install it eventually.

```bash
gem install rails -v 2.3.11
```



2. ### MySQL

The software we will be using requires version 5 of mysql. For operating systems other than MacOS, or details/alternatives, refer to [the official installation guide](https://dev.mysql.com/doc/refman/5.7/en/installing.html). 

MacOS users can use the following brew commands to install MySQL:

```bash
brew install mysql@5.7
```

Once installed MySQL needs to be added to path so when we run `mysql` in the terminal, the correct program is located and initiated.

```bash
export PATH=$PATH:$(brew --prefix mysql@5.7)/bin
```

We will also add it our `.bashrc` file so we can access it in future sessions. If you use a different shell, such as `zsh`, you may consider appending this to `~/.bashrc` instead.

```bash
echo "export PATH=\$PATH:$(brew --prefix mysql@5.7)/bin" >> ~/.bashrc
```



Once installed, mysql needs to be started. If you installation method was different from brew, your command to start may be `mysql.server start`.

```bash
brew services start mysql@5.7
```



We now have a database server running on our computer. However, we still need to configure it properly to be usable by the application. 

```bash
# launch mysql
mysql -u root 
```

You should now have new "prompt" that looks something like `mysql>`. The following commands are meant to be passed to `mysql`, if the `mysql>` prompt did not appear, you may need to do some troubleshooting. 

```mysql
# ignore the `mysql>`
mysql> create database turkopticon_devel;
```

The `your_password` in the following command can be changed to anyting you like. However, you should keep it simple, or write it down because you will soon need to enter it into a file for the application.

```mysql
# feel free to choose your own password here
# this command creates the "user" which is how the application will interface with the database
mysql> create user 'turkopticon_dev'@'localhost' identified by 'your_password';
```

```mysql
# this command gives many accesses to our application
mysql> grant all privileges on turkopticon_devel.* to 'turkopticon_dev'@'localhost';
```

```bash
# exit mysql with ctrl+d
# after pressing ctrl+d, you should see the 'mysql>' prompt disappear
```

```bash
# the following command will set up all of the tables that are needed
mysql -u root turkopticon_devel < ./db/turkopticon-empty-structure.sql
```



3. ### Gems

```bash
gem install fastthread -v 1.0.7
```

```bash
gem install mysql -v 2.9.1
```

```bash
gem install json -v 1.8.1
```

```bash
gem install haml -v 3.1.2
```

```bash
gem install rubygems-update -v 1.3.7
```

```bash
update_rubygems --version=1.3.7
```

```bash
# this gem is installed from a local file
gem install --local tmp/gem/mislav-will_paginate-2.3.11.gem --no-rdoc --no-ri
```

```bash
# you can skip this one if you installed it earlier
gem install rails -v 2.3.11
```




4. ### Local Env Files

We now need to create a few config files that are specific to your dev environment.

- environment.rb

  ````bash
  # this takes us into the config directory
  cd config
  ````

  ```bash
  # make a copy of the example
  cp environment-example.rb environment.rb
  ```

  Then open `environment.rb` and replace `secret` with any number that has more than 30 digits. Feel free to mash those number keys, and get some frustration of installing all these dependencies manually out of your system. You're almost there.

- database.yml

  ```bash
  # make a copy of the example
  cp database-example.yml database.yml
  ```

  Then open `databse.yml` and on line 6, replace `'your_password'` with the password you chose in step 2 (MySQL).
  
  ```bash
  # exit this directory
  cd ..
  ```
  
  

5. ### Launch the App!

You should now be able to start the application. You can run this by going into the `script` directory and running the command `./server start`. Visit [localhost:3000](localhost:3000) to see the website!

```bash
cd script
```

```bash
./server start
```



