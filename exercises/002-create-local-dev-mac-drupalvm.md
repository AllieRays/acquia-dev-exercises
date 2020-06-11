# Create a local development Environment on a Mac OS with DrupalVm and Acquia BLT.

# System Requirements 

() Git \
() Composer \
() PHP 7.3 \
() Homebrew  \
() Xcode  \
() mac0s 10.9 or greater \
() Vagrant 2.2.5  [download](https://releases.hashicorp.com/vagrant/2.2.9/vagrant_2.2.9_x86_64.dmg) \
() Virtualbox Version 6.0.12 r133076 (Qt5.6.3) [download](https://download.virtualbox.org/virtualbox/6.0.12/VirtualBox-6.0.12-133076-OSX.dmg) \
() pip \
() Ansible 2.7 

### Networking considerations
Building project dependencies requires your local computer to make HTTP and HTTPS requests to several remote software providers. Your local- and network-level security settings must not block requests.

## Step One: Check Prerequisites
Check you have the following by checking their version numbers  \
() Git `git --version`  \
() Composer  `composer --version`  \
() PHP 7.3   `php --version`  \
() Homebrew `brew --verison`  \
() Xcode ` xcodebuild -version`  \
() mac0s 10.9 or greater `sw_vers`  \
() Vagrant `vagrant --version`  \
() Virtualbox `virtualbox --version` \
() Ansible 2.7  `ansible --version` 
```
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ sudo python get-pip.py
$ sudo pip install ansible
```
If you are installing on macOS Mavericks (10.9), you may encounter some noise from your compiler. A workaround is to do the following:
```
$ CFLAGS=-Qunused-arguments CPPFLAGS=-Qunused-arguments pip install --user ansible`
```

## Step Two: xcode 
Ensure that you have installed Xcode. Xcode is required to support Homebrew, and you can install Xcode on macOS 10.9 or greater by running the following commands:

```
$ sudo xcodebuild -license
$ xcode-select --install
```

## Step Three: Install the minimum dependencies for Acquia BLT. 
Although you can use the following commands to use Homebrew to install the needed packages, you are not required to use a package manager:

```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew install php git composer
$ composer global require hirak/prestissimo:^0.3
$ composer global require zaporylie/composer-drupal-optimizations:^1.1
```

## Step Four: Fork the project repository in GitHub. 
Fork your repo. [Read this for more info about how to fork.](https://help.github.com/en/github/getting-started-with-github/fork-a-repo) 

## Step Five: Request access to the Acquia Cloud Environment for your project.
Please reach out to an Acquian technical resource if needed.

## Step Six: Create an SSH key that can be used for GitHub
Open your terminal and create an ssh key. \
`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` \
() type yes and do not save with a passphrase \
() Cat and copy your key `/Users/[username]/.ssh/id_rsa` \
() Add your key to your github account. \
... () Go to https://github.com/settings/keys and add a new ssh key \
....() save the key on github \
....() in your terminal type in `ssh-add -K ~/.ssh/id_rsa`

## Step Seven: Clone your forked repository. By default, Git names this "origin" on your local.
This project is based on BLT, an open-source project template and tool that enables building, testing, and deploying Drupal installations following Acquia Professional Services best practices. While this is one of many methodologies, it is our recommended methodology. 
Clone your forked repository. By default, Git names this "origin" on your local.
```
$ git clone git@github.com:[github-username]/Santa-Clara-County.git
```
To ensure that upstream changes to the parent repository may be tracked, add the upstream locally as well.
```
$ git remote add upstream git@github.com:acquia-pso/Santa-Clara-County.git
```

Check out the `dev-assessment-exercises` branch
```
$ git checkout dev-assessment-exercises
```

## Step Eight: Install dependencies 
BLT provides an automation layer for testing, building, and launching Drupal 8 applications. For ease when updating codebase it is recommended to use  Drupal VM. If you prefer, you can use another tool such as Docker, [DDEV](https://docs.acquia.com/blt/install/alt-env/ddev/), [Docksal](https://docs.acquia.com/blt/install/alt-env/docksal/), [Lando](https://docs.acquia.com/blt/install/alt-env/lando/), (other) Vagrant, or your own custom LAMP stack, however support is very limited for these solutions. \
Install Composer dependencies. \
After you have forked, cloned the project and setup your blt.yml file install Composer Dependencies. (Warning: this can take some time based on internet speeds.)
```
$ composer install
```

If composer gives you trouble, remove any downloaded directories and increase your memory limit for composer. 
```
ulimit -n 10000
rm -rf docroot/core
 rm -rf vendor
 php -d memory_limit=-1 /usr/local/bin/composer install
```

## Step Nine: Ensure BLT is installed as a command. 
Manually add this BLT function to your .bashrc file
```
nano  ~\.bashrc
```
add this function

```
function blt() {
  if [ "'git rev-parse --show-cdup 2> /dev/null'" != "" ]; then
    GIT_ROOT=$(git rev-parse --show-cdup)
  else
    GIT_ROOT="."
  fi

  if [ -f "$GIT_ROOT/vendor/bin/blt" ]; then
    $GIT_ROOT/vendor/bin/blt "$@"
  else
    echo "You must run this command from within a BLT-generated project repository."
  fi
}
```

```
source ~/.bashrc
```

## Step Ten: Setup VM.
Setup the VM with the configuration from this repositories [configuration files](#important-configuration-files). 
```
$ blt vm
$ vagrant up
```

## Step Eleven: SSH into your VM.
SSH into your localized Drupal VM environment automated with the BLT launch and automation tools.
```
$ vagrant ssh
```

## Step Twelve: SSH into your VM.
Setup a local Drupal site with an empty database.  \
Use BLT to setup the site with configuration.
```
$ blt setup
```

## Step Thirteen: Log into your site with drush. 
Access the site and do necessary work at http://local.SITENAME.com by running the following commands.
```
$ cd docroot
$ drush uli
```
---

# More Resources 
() Review https://github.com/acquia-pso/Santa-Clara-County#my-project for more information.  
() Review the [Required / Recommended Skills](https://docs.acquia.com/blt/developer/skills/) for working with a BLT project.



## NFS troubleshooting. 
NSF wont mount

() virtualbox 6.1.8
() system preferences -> security & privacy -> full disk access unlock + command+shift+. sbin/nfsd
() `sudo rm /etc/exports`
() `vagrant up --provision`
() path is outside of Documents or Downloads /Users/user-name/sites/Santa-Clara-County
() tried to restart nfsd `sudo nfsd restart`
() Updated the Vagrantfile with the following
() added `vagrant plugin install vagrant-vbguest`
```
# Add after this line
host_project_dir = ENV['DRUPALVM_PROJECT_ROOT'] || host_drupalvm_dir

if Dir.exist?('/System/Volumes/Data')
    host_project_dir  = '/System/Volumes/Data' + host_project_dir
end
```



to try next
`nfsd checkexports`
