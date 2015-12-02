source 'https://rubygems.org'

gem 'jekyll'
gem 'jekyll-sitemap'
gem 'jekyll-import'
gem 'sequel'
gem 'redcarpet'


# ensures the correct version of github pages always
require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
