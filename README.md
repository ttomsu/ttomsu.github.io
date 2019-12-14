# Install RVM ([source](https://github.com/rvm/ubuntu_rvm)):

1. Get RVM:
	```
	sudo apt-get install software-properties-common
	sudo apt-add-repository -y ppa:rael-gc/rvm
	sudo apt-get update
	sudo apt-get install rvm
	```

2. Gnome Terminal profile --> "Run command as a login shell"

3. Reboot

4. Execute:
	```
	rvm install ruby
	gem install bundler
	bundle install
	bundle exec jekyll serve --watch --incremental
	```