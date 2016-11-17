# Alan Zanotti

Personal site for blogging and showing off in general. Built using the [Jekyll](https://jekyllrb.com/)
static site generator.

## Setting Up a DEV / Build Environment

Jekyll requires Ruby to build a site. These instructions install Ruby Version Manager (rvm)
along with Ruby and any other supporting softwares.

```
#!/bin/bash

ruby_version=$1

if [ -z $ruby_version ]
then
	echo "Usage: ./setup-development-environment.sh <RUBY_VERSION>"
	exit -1
fi

# Install public signing key
command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

# Download and run the RVM installation script
curl -sSL https://get.rvm.io | bash -s stable

# Append the following to the user's .bash_profile
echo "source ~/.profile" >> $HOME/.bash_profile
source $HOME/.bash_profile

# Install the specified version of ruby
rvm install $ruby_version

# Set the default for new shells
rvm use $ruby_version --default

# Echo the version for good measure
ruby -v

# Install bundler
gem install bundler

exit 0
```

## Building the Site

For development:
```
jekyll build --drafts
```

For __PRODUCTION__:
```
JEKYLL_ENV="production" jekyll build
```

The production build places the Google Analytics code into the footer.
