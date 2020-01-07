---
title: Layout
author: Mario
layout: article
show_title: true
show_tags: true
aside:
  toc: true
---

## [Haml](http://haml.info) setup
* Add `gem 'haml-rails'` to your `Gemfile`
* After running `bundle install` convert the layout and all the other template files into the corresponding Haml version - for details see the [GitHub repository](https://github.com/haml/haml-rails): `HAML_RAILS_DELETE_ERB=true rails haml:erb2haml`

## [Boostrap](https://getbootstrap.com) setup
* Background information can be found in following [tutorial](https://rossta.net/blog/webpacker-with-bootstrap.html)
* Install the bootstrap package plus necessary dependencies: `yarn add bootstrap jquery popper.js`
* Setup the site stylesheet and import bootstrap by creating a new file `app/javascript/stylesheets/application.scss` with following content:
    ```scss
    @import 'bootstrap_custom';
    @import '~bootstrap/scss/bootstrap';
    @import 'site';
    ```
  * the `app/javascript/stylesheets/_bootstrap_custom.scss` file will contain all the styling adjustments based on bootstrap variables, e.g. to set a custom font:
      ```scss
      @import url('https://fonts.googleapis.com/cssfamily=Lato:300,300i,400,400i,500,500i,700,700i&display=swap');
      @import url('https://fonts.googleapis.com/cssfamily=Anonymous+Pro:300,300i,400,400i,500,500i,700,700i&display=swap');
      
      $font-family-sans-serif: Lato, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'Noto Sans', sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji' !default;
      $font-family-monospace: 'Anonymous Pro', SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace !default;
      
      $font-size-base: 1.25rem;
      ```
  * the `app/javascript/stylesheets/_site.scss` file will contain all other styling adjustments, e.g. the necessary adjustment for the fixed header - see application layout definition below:
      ```scss
      .hvworkout {
        // using fixed top navigation - therfore a padding for <main> is necessary
        &__has-fixed-top-header { padding-top: 4.5rem; }
      }
      ```
* Adjust the application pack `app/javascript/packs/application.js` to look like:
    ```javascript
    ...
    // Import application specific stylesheets
    import '../stylesheets/application'
    
    // Activate bootstrap
    import 'jquery'
    import 'popper.js'
    import 'bootstrap'
    import '../_bootstrap_custom.js'
    ...
    ```
  * Enable bootstrap _popover_ and _tooltips_ by creating a new file  `app/javascript/_bootstrap_custom.js`:
      ```javascript
      $(function() {
        $('[data-toggle="tooltip"]').tooltip()
      })
   
      $(function() {
        $('[data-toggle="popover"]').popover()
      })
      ```
* Webpack configuration `config/webpack/environment.js` has to be adjusted too:
    ```ruby
    const { environment } = require('@rails/webpacker')
    
    const webpack = require('webpack')
    environment.plugins.append(
      'Provide',
      new webpack.ProvidePlugin({
        $: 'jquery',
        jQuery: 'jquery',
        Popper: ['popper.js', 'default']
      })
    )
    
    module.exports = environment
    ```
* Change the content of the `app/views/layout/application.html.haml` to look like:
    ```haml
    !!!
    %html{lang: 'en'}
      %head
        %meta{'http-equiv': 'Content-Type', content: 'text/html; charset=UTF-8'}/
        %meta{name: 'viewport', content: 'width=device-width, initial-scale=1.0'}/
        %title= content_for?(:title) ? yield(:title) : HVWorkout
        %meta{name: 'description', content: "#{content_for?(:description) ? yield(:description) : 'HVWorkout - Tracking Kieser Training exercises'}"}/
        = csrf_meta_tags
        = csp_meta_tag
        = stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload'
        = javascript_pack_tag 'application', 'data-turbolinks-track': 'reload'
    
      %body.hvworkout
        %header
          = render 'layouts/navigation'
        %main.container.hvworkout__has-fixed-top-header
          = yield
    ```
  * The navigation partial `app/views/layout/_navigation.html.haml` defines a standard header fixed at the top:
      ```haml
      %nav.navbar.fixed-top.navbar-expand-md.navbar-light.bg-light
        .container
          = link_to 'HVboom - HVWorkout', root_path, class: 'navbar-brand'
          %button.navbar-toggler{'data-target': '#navbar', 'data-toggle': 'collapse', 'aria-controls': 'navbar', 'aria-expanded': 'false', 'aria-label': 'Toggle navigation', type: 'button'}
            %span.navbar-toggler-icon
          #navbar.collapse.navbar-collapse
            %ul.navbar-nav.ml-auto
              %li.nav-item
                = link_to 'Home', home_path, class: 'nav-link'
              %li.nav-item= link_to 'About', about_path, class: 'nav-link'
      ```
			

## Static pages
* Ideas taken from following [tutorial](https://www.learnenough.com/ruby-on-rails-4th-edition-tutorial/static_pages#sec-static_pages)
* Create the static pages controller: `rails generate controller StaticPages home about`
* Adjust the `config/routes.rb` to look like:
    ```ruby
    Rails.application.routes.draw do
      # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
      root 'static_pages#home'
      get '/home', to: 'static_pages#home'
      get '/about', to: 'static_pages#about'
    end
    ```