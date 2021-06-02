### Overview
Simon Krueger's personal blog.

View the website at http://www.simonkrueger.com


### Development
The site is built using [Jekyll](https://jekyllrb.com/).

[Ubuntu installation](https://jekyllrb.com/docs/installation/ubuntu/)

```sh
sudo apt-get install ruby-full build-essential zlib1g-dev

echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

gem install jekyll bundler
```

Might also need to install `jekyll-paginate`

```sh
gem install jekyll-paginate
```



Build the site and make it available on a local server.

```sh
jekyll serve --livereload
```

(might need `bundle exec jekyll` depending on if jekyll is on the path)

```example
➜ which jekyll
/home/simon/gems/bin/jekyll
```

Browse to http://localhost:4000

