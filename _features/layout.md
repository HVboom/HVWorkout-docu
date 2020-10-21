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

    $enable-responsive-font-sizes: true;
    ```
  * the `app/javascript/stylesheets/_site.scss` file will contain all other styling adjustments, e.g. helper classes to show screen size dependent content or the necessary adjustment for the fixed header - see application layout definition below:
    ```scss
    .hvworkout {
      // define utility classes to hide content on smaller screens
      &__d-xl-only,
      &__d-lg-only,
      &__d-md-only,
      &__d-sm-only,
      &__d-xs-only {
        display: none;
      }   
      @include media-breakpoint-only(xl) {
        &__d-xl-only { display: initial; }
      }   
      @include media-breakpoint-only(lg) {
        &__d-lg-only { display: initial; }
      }   
      @include media-breakpoint-only(md) {
        &__d-md-only { display: initial; }
      }   
      @include media-breakpoint-only(sm) {
        &__d-sm-only { display: initial; }
      }   
      @include media-breakpoint-only(xs) {
        &__d-xs-only { display: initial; }
      }   

      // using fixed top navigation - therefore a padding for <main> is necessary
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
  %html
    %head
      %meta{'http-equiv': 'content-type', content: 'text/html; charset=UTF-8'}/
      %meta{name: 'viewport', content: 'width=device-width, initial-scale=1.0, shrink-to-fit=no'}/
      %meta{'http-equiv': 'content-language', content: "#{locale}"}/
      %title= content_for?(:title) ? yield(:title) : HVWorkout
      %meta{name: 'description', content: "#{content_for?(:description) ? yield(:description) : 'HVWorkout - Tracking Kieser Training exercises'}"}/
      = csrf_meta_tags
      = csp_meta_tag
      = stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload'
      = javascript_pack_tag 'application', 'data-turbolinks-track': 'reload'
  
    %body.hvworkout{class: "#{controller.controller_name}"}
      %header
        = render 'layouts/navigation'
      %main.container-md.hvworkout__has-fixed-top-header{role: 'main'}
        = render 'layouts/messages'
        = yield
  ```
  * The navigation partial `app/views/layout/_navigation.html.haml` defines a standard header fixed at the top:
    ```haml
    %nav.navbar.fixed-top.navbar-expand-md.navbar-light.bg-light
      .container-md
        = render 'layouts/brand'
        %button.navbar-toggler{data: {target: '#navbar', toggle: 'collapse'}, aria: {controls: 'navbar', expanded: 'false', label: 'Toggle navigation'}, type: 'button'}
          %span.navbar-toggler-icon
        #navbar.collapse.navbar-collapse
          %ul.navbar-nav.ml-auto
            %li.nav-item
              = link_to home_path, class: 'nav-link' do
                = fa_icon 'home'
                Home
            %li.nav-item
              = link_to about_path, class: 'nav-link' do
                = fa_icon 'address-card'
                About
    ```
  * The brand partial `app/views/layout/_brand.html.haml` defines an applications branding, which is optimized for different screen sizes:
      ```haml
      = link_to root_path, class: 'navbar-brand' do
        = image_pack_tag 'Stars.svg', class: 'hvworkout__brand-img'
        %span.hvworkout__d-xl-only.hvworkout__d-lg-only HVboom - HVWorkout
        %span.hvworkout__d-md-only.hvworkout__d-sm-only HVboom
      ```
  * The messages partial `app/views/layout/_messages.html.haml` defines a the display of the flash messages:

      ```haml
      -# Rails flash messages styled for Bootstrap 4.0
      - flash.each do |name, msg|
        - if msg.is_a?(String)
            .alert{class: "alert-#{name.to_s == 'notice' ? 'success' : 'danger'}", role: 'alert'}
                %button.close{'aria-hidden': 'true', 'data-dismiss': 'alert', type: 'button'} &times;
                = content_tag :div, msg, id: "flash_#{name}"
      ```


## [Fontawesome](https://github.com/tomkra/font_awesome5_rails#3-install-with-webpack) setup
* Add the `gem 'font_awesome5_rails'` and follow the [instructions](https://hackernoon.com/integrate-bootstrap-4-and-font-awesome-5-in-rails-6-u87u32zd) to add the package by calling `yarn add @fortawesome/fontawesome-free`
* Activate fontawsome styles `app/javascript/stylesheets/application.scss`:
  ```scss
  @import 'bootstrap_custom';
  @import '~bootstrap/scss/bootstrap';
  @import '@fortawesome/fontawesome-free';
  @import 'site';
  ```
* Activate fontawsome to be used in javascripts in file `app/javascript/packs/application.js`:
  ```javascript
  ...
  // Activate fontawesome
  import '@fortawesome/fontawesome-free/js/all';
  ...
  ```


## Favicon
* Ideas taken from following [tutorial](https://medium.com/tech-angels-publications/bundle-your-favicons-with-webpack-b69d834b2f53)
* Use the online tool [RealFaviconGenerator.net](https://realfavicongenerator.net/favicon/ruby_on_rails) to generate all necessary files to have favicons optimized for all different browsers.
*  Option 1 - use the generated files and follow the tutorial
    * Copy the generated files into `app/javascript/favicons/`
    * Create a script `app/javascript/favicons/favicon.js` to copy these files into the public target directory
      ```javascript
      const faviconsContext = require.context(
        '!!file-loader?name=media/favicons/[name].[ext]!.',
        true,
        /\.(svg|png|ico|xml|webmanifest)$/
      );
      faviconsContext.keys().forEach(faviconsContext);
      ```
		* Include that script in `app/javascript/packs/application.js`
		  ```javascript
      ...
      // Copy favicon related files
      import '../favicons/favicon'
      ...
      ```
    * Create a partial to be included into the head part of the layout `app/views/layout/_favicon.html.haml`:
      ```haml
      %link{href: "#{asset_pack_path 'media/favicons/apple-touch-icon.png'}", rel: "apple-touch-icon", sizes: "180x180"}/
      %link{href: "#{asset_pack_path 'media/favicons/favicon-32x32.png'}", rel: "icon", sizes: "32x32", type: "image/png"}/
      %link{href: "#{asset_pack_path 'media/favicons/favicon-16x16.png'}", rel: "icon", sizes: "16x16", type: "image/png"}/
      %link{href: "#{asset_pack_path 'media/favicons/favicon.ico'}", rel: "shortcut icon"}/
      %link{href: "#{asset_pack_path 'media/favicons/safari-pinned-tab.svg'}", rel: "mask-icon", color: "#5bbad5"}/
      %link{href: "#{asset_pack_path 'media/favicons/site.webmanifest'}", rel: "manifest"}/
      %meta{content: "HVWorkout", name: "apple-mobile-web-app-title"}/
      %meta{content: "HVWorkout", name: "application-name"}/
      %meta{content: "#ffffff", name: "theme-color"}/
      %meta{content: "#2b5797", name: "msapplication-TileColor"}/
      %meta{content: "#{asset_pack_path 'media/favicons/browserconfig.xml'}", name: "msapplication-config"}/
      ```
      * Include that partial in `app/views/layout/application.html.haml`
        ```haml
        !!!
        %html{lang: 'en'}
          %head
            ...
            = render 'layouts/favicon'
            ...
          
          %body.hvworkout
          ...
        ```
  * Option 2 - use the gem file and follow the instructions provided for Ruby On Rails
      * Remark: I had to increase the timeout in file `rails_real_favicon-0.0.13/lib/generators/favicon_generator.rb`:
        ```ruby
        ...
        resp = Net::HTTP.start(uri.host, uri.port, use_ssl: uri.scheme == "https", read_timeout: 200) do |http|
        ...
        ```
      * My configuration file look like:
        ```json
        {
          "master_picture": "app/javascript/images/Stars.svg",
          "favicon_design": {
            "ios": {
              "picture_aspect": "no_change",
              "assets": {
                "ios6_and_prior_icons": false,
                "ios7_and_later_icons": true,
                "precomposed_icons": true,
                "declare_only_default_icon": true
              }
            },
            "desktop_browser": [
        
            ],
            "windows": {
              "picture_aspect": "no_change",
              "background_color": "#da532c",
              "on_conflict": "override",
              "assets": {
                "windows_80_ie_10_tile": false,
                "windows_10_ie_11_edge_tiles": {
                  "small": false,
                  "medium": true,
                  "big": false,
                  "rectangle": false
                }
              }
            },
            "android_chrome": {
              "picture_aspect": "no_change",
              "theme_color": "#ffffff",
              "manifest": {
                "name": "HVWorkout",
                "start_url": "https:\/\/hvworkout.demo.hvboom.org",
                "display": "standalone",
                "orientation": "not_set",
                "on_conflict": "override",
                "declared": true
              },
              "assets": {
                "legacy_icon": false,
                "low_resolution_icons": false
              }
            },
            "safari_pinned_tab": {
              "picture_aspect": "black_and_white",
              "threshold": 76.09375,
              "theme_color": "#5bbad5"
            }
          },
          "settings": {
            "scaling_algorithm": "Mitchell",
            "error_on_image_too_small": false,
            "readme_file": false,
            "html_code_file": true,
            "use_path_as_is": false
          }
        }
        ```
      * *Attention*: the generated file `config/favicon.yml` does not work with Rails 6, therefore you have to adapt the files more or less like for option 1


## Static pages
* Ideas taken from following [tutorial](https://www.learnenough.com/ruby-on-rails-4th-edition-tutorial/static_pages#sec-static_pages)
* Create the static pages controller: `rails generate controller StaticPages home about`
* Adjust the `config/routes.rb` to look like:
  ```ruby
  Rails.application.routes.draw do
    root 'static_pages#home'
    get '/home', to: 'static_pages#home'
    get '/about', to: 'static_pages#about'
  end
  ```
		
## [Simple Form](http://simple-form-bootstrap.plataformatec.com.br) setup
* A great gem to produce really nice looking forms with low effort
* After adding the `gem simple_form` to your `Gemfile` run the script to install it `rails generate simple_form:install --bootstrap`
* Adjust the form template to use floating labels - `lib/templates/haml/scaffold/_form.html.haml`
  ```ruby
  -# frozen_string_literal: true
  = simple_form_for @<%= singular_table_name %> do |f|
    = f.error_notification
    = f.error_notification message: f.object.errors[:base].to_sentence if f.object.errors[:base].present?

    .form-inputs
      - attributes.each do |attribute|
        = f.<%= attribute.reference? ? :association : :input %> :<%= attribute.name %>,
          required: true,
          autofocus: true,
          input_html: { autocomplete: '<%= attribute.name %>', placeholder: true }

    .form-actions
      = render 'shared/actions', form: f, submit: 'Submit'
  ```
  * To complete the setup for floating labels I had to extract the stylesheets `https://getbootstrap.com/docs/4.3/examples/floating-labels/` into the file `app/javascript/stylesheets/_form_floating_labels.scss`. That stylesheet has to be imported in `app/javascript/stylesheets/application.scss`.
* The template uses a partial to define standard form actions - `app/views/shared/_actions.html.haml`

  ```ruby
  - back ||= back.nil?

  = form.button :submit, submit, class: 'btn-primary'
  = form.button :button, 'Reset', type: :reset, class: 'btn-outline-secondary'
  -if back
    = link_to 'Back', :back, class: 'btn btn-outline-secondary float-right'
  ```

  * That partial is called with mandatory local variables for the `form` and `submit` button label. And you can provide an optional boolean `back` to have a back button rendered.


## [Gravatar](https://en.gravatar.com/site/implement/images/)
* Show an Avatar of the logged in user.
* It's implemented as a generic service allowing to be called with any object responding to the `#email` method or with a string containing the email.
* In addition the `:size` of the returned image can be specified in the option hash. The default is 128 pixels.
* Complete source code - `app/services/gravatar.rb`:
  ```ruby
  class Gravatar
    # https://en.gravatar.com/site/implement/images/
 
    EMAIL =           :email
    HOST =            'https://secure.gravatar.com/avatar/'
    IMAGE_TYPE =      '.png'
    GUEST =           'guest@hvboom.ch'
    RATING =          'g'
    DEFAULT =         'mp'
    SIZE =            '128'
    ALLOWED_OPTIONS = [:size]
 
    class << self
      def url(user_or_email = GUEST, options = {})
        base_url(user_or_email) + '?' + sanitize_options(options).to_query
      end
 
 
      private
 
      def base_url(user_or_email)
        HOST + gravatar_id(email(user_or_email)) + IMAGE_TYPE
      end
 
      def email(user_or_email)
        GUEST if user_or_email.blank?
 
        if user_or_email.respond_to?(EMAIL)
          user_or_email.send(EMAIL)
        else
          user_or_email.to_s
        end
      end
 
      def gravatar_id(email)
        Digest::MD5::hexdigest(email.strip.downcase)
      end
 
      def sanitize_options(options)
        { 
          rating:  RATING,
          default: DEFAULT,
          size:    SIZE
        }.merge(options.extract!(ALLOWED_OPTIONS))
      end
    end
  end
  ```
* To support a smooth integration into the appliction I created following helper module - `app/helpers/gravatar_helper.rb`
  ```ruby
  module GravatarHelper
    # https://mixandgo.com/learn/the-beginners-guide-to-rails-helpers
 
    def user_gravatar_tag(user)
      image_tag "#{Gravatar.url(user)}", alt: '', class: 'hvworkout__user-img'
    end
 
    def guest_gravatar_tag
      image_tag "#{Gravatar.url()}", alt: '', class: 'hvworkout__user-img'
    end
 
    def profile_gravatar_tag(user)
      content_tag :div, class: 'hvworkout__profile-gravatar' do
        link_to 'https://secure.gravatar.com/', target: '_blank', rel: 'noopener noreferrer' do
          (image_tag "#{Gravatar.url(user)}", alt: '', class: 'hvworkout__profile-gravatar-img') +
          (content_tag :div, class: 'hvworkout__profile-gravatar-link' do
            (content_tag :i, '', class: 'fas fa-edit').html_safe +
            (content_tag :span, 'Edit', class: 'sr-only') + ' on Gravatar.com'
          end)
        end
      end
    end
  end
  ```
