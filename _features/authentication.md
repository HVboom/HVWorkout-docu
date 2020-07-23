---
title: Authentication
author: Mario
layout: article
show_title: true
show_tags: true
aside:
  toc: true
---

## Setup [Devise](https://github.com/plataformatec/devise)
* Background information can be found in following [tutorial](https://iridakos.com/programming/2019/04/04/creating-chat-application-rails-websockets)
* Install the devise package: `rails generate devise:install`
  * Configure the mail delivery for the confirmation feature - `config/environments/development.rb` and `config/environments/production.rb`:
    ```ruby
    config.action_mailer.raise_delivery_errors = true
    config.action_mailer.perform_deliveries = true
    config.action_mailer.default_url_options = { host: 'hvworkout.demo.hvboom.org', protocol: 'https://' }
    ```
  * Adjust the standard settings in `config/initializers/devise.rb`:
    * Define the sender address: `config.mailer_sender = 'HVWorkout@HVboom.ch'`
    * Enable the turbolinks configuration:
      ```ruby
      # ==> Turbolinks configuration
      # If your app is using Turbolinks, Turbolinks::Controller needs to be included to make redirection work correctly:
      #
      ActiveSupport.on_load(:devise_failure_app) do
        include Turbolinks::Controller
      end
      ```
* Create an extended user model: `rails generate devise User nickname:string`
* Adjust the user model to use additional devise components and to validate the nickname:
    ```ruby
    class User < ApplicationRecord
      # Include default devise modules. Others available are:
      # :lockable, :timeoutable and :omniauthable
      devise :database_authenticatable, :registerable,
             :recoverable, :rememberable, :validatable,
             :confirmable, :trackable
 
      validates :nickname, uniqueness: { case_sensitive: false }, presence: true
    end
    ```
* An adjustment to the db migration for the user model has to be done to reflect the model modifications:
  * set defaults for the nickname
  * enable the columns for the trackable and confirmable feature
    ```ruby
    # frozen_string_literal: true    
    class DeviseCreateUsers < ActiveRecord::Migration[6.0]
      def change
        create_table :users do |t| 
          ## Database authenticatable
          t.string :email,              null: false, default: ''
          t.string :nickname,           null: false, default: ''
          t.string :encrypted_password, null: false, default: ''
    
          ## Recoverable
          t.string   :reset_password_token
          t.datetime :reset_password_sent_at
    
          ## Rememberable
          t.datetime :remember_created_at
    
          ## Trackable
          t.integer  :sign_in_count, default: 0, null: false
          t.datetime :current_sign_in_at
          t.datetime :last_sign_in_at
          t.inet     :current_sign_in_ip
          t.inet     :last_sign_in_ip
    
          ## Confirmable
          t.string   :confirmation_token
          t.datetime :confirmed_at
          t.datetime :confirmation_sent_at
          t.string   :unconfirmed_email # Only if using reconfirmable
    
          ## Lockable
          # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
          # t.string   :unlock_token # Only if unlock strategy is :email or :both
          # t.datetime :locked_at
    
          t.timestamps null: false
        end 
    
        add_index :users, :email,                unique: true
        add_index :users, :nickname,             unique: true
        add_index :users, :reset_password_token, unique: true
        add_index :users, :confirmation_token,   unique: true
        # add_index :users, :unlock_token,         unique: true
      end 
    end 
    ```
* Enforce login before the application can be used by changing the application controller `app/controller/application_controller.rb`:
  ```ruby
  class ApplicationController < ActionController::Base
    before_action :authenticate_user!
    before_action :configure_permitted_parameters, if: :devise_controller?
 
    protected
 
    def configure_permitted_parameters
      devise_parameter_sanitizer.permit(:sign_up, keys: [:email, :nickname])
    end
  end
  ```
  * You can explicitly skip the authentication for specific actions like in the `app/controller/static_pages_controller.rb`:
    ```ruby
    class StaticPagesController < ApplicationController
      skip_before_action :authenticate_user!, only: [:about]

      ...
    ```
    It is a denylist approach - every access to the application needs to be done with a known user as long as the access is not explicitly allowed.
* Create views to be able to add the `nickname` attribute:
  ```bash
  rails generate devise:views
  HAML_RAILS_DELETE_ERB=true rails haml:erb2haml
  ```
  See files in the `app/views/devise/` directory.
* The current user layout is handled in the partial `app/views/layouts/_authentication.html.haml` which is included into the navigation `app/views/layouts/_navigation.html.haml`:

  ```ruby
  - if current_user
    .navbar-nav.nav-item.dropdown
      %a#userDropdown.nav-link.dropdown-toggle{aria: {expanded: 'false', haspopup: 'true'}, data: {toggle: 'dropdown'}, href: '#', role: 'menu'}
        = user_gravatar_tag current_user
        = current_user.nickname
      .dropdown-menu{aria: {labelledby: 'userDropdown'}}
        = link_to edit_registration_path(current_user), class: 'dropdown-item' do
          = fa_icon 'user'
          Profile
        .dropdown-divider
        = link_to destroy_user_session_path, method: :delete, class: 'dropdown-item' do
          = fa_icon 'sign-out-alt'
          Log out
  ```
  * Here is the final result of the user profile page:
    ![User Profile]({{ site.images }}UserProfile.png){:.border.rounded}
* Seed the DB for the demo application:
  ```ruby
  # This file should contain all the record creation needed to seed the database with its default values.
  # The data can then be loaded with the rails db:seed command (or created alongside the database with db:setup).
  unless Rails.env.production?
    # create Demo users
    3.times do |n|
      nickname = "Demo#{n+1}"
      unless User.find_by_nickname(nickname)
        User.create!(nickname: nickname, email: "#{nickname}@HVboom.ch", password: 'Demo2020', password_confirmation: 'Demo2020', confirmed_at: DateTime.now)
        puts "User '#{nickname}' created"
      end
    end
  end
  ```
