#ruby=ruby-2.6.5
#ruby-gemset=HVWorkout
#

source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end

gem 'jekyll' # , '4.1.1'

# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "jekyll-text-theme"

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem 'jekyll-feed'
  gem 'jekyll-admin' # , '0.10.2'
  gem 'jekyll-target-blank'
  # gem 'jekyll-plantuml'
end

# avoid polling of changed files
require 'rbconfig'
if RbConfig::CONFIG['target_os'] =~ /(?i-mx:bsd|dragonfly)/
  gem 'rb-kqueue' # , '>= 0.2'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]

