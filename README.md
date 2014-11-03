## Google Maps with Ruby on Rails

Google Maps is an amazing api that can give an app a ton of added functionality with a very small ammout of code.

In this article I will show you what is necessary to get it running with Ruby on Rails and display your users on a map.

We will use 2 gems to accomplish this:

      Google maps for Rails-
      https://github.com/apneadiving/Google-Maps-for-Rails
      
      Geocoder-
      https://github.com/alexreisner/geocoder
            
  So how do Rails, the map, the geocoder, and your database fit together? Google maps for rails will allow us to embed a map in a rails view with a bit of embedded javascript. The geocoder in our users model will take an address which we will build from the various address fields a user fills out when they sign up. It will then give us a latitude and longitude which will be stored in their own columns in our users table. We will use those values to make markers which will represent each user on the map!

First off we need to install the gems listed above. For this simply add to your gemfile:

```ruby
      gem "gmaps4rails"
      gem "geocoder"
```

Then remember to bundle install.

Next we need to get the map up and running. For this we will pick a view to hold the map and place the following JavaScript in the file. One thing to note is I was unable to get this working when I linked the JS from a seperate file. It has to be embedded in the view for this to work.

```javascript
      <script type='text/javascript'>
      handler = Gmaps.build('Google');
      handler.buildMap({
          provider: {
            disableDefaultUI: false
          },
          internal: {
            id: 'map'
          }
        },
        function(){
          markers = handler.addMarkers(<%=raw @hash.to_json %>);
          handler.bounds.extendWith(markers);
          handler.fitMapToBounds();
        }
      );
      </script>
```

To tell the script where to display the map simply add an empty div element to the file with an id of map like so:

```html
      <div id="map">
      </div>
```
      
There are more options in the Google Maps for Rails readme if you need them but this will do for a basic map. Don't worry about the markers for now. We'll get to those in a second. 

The map:
![Basic map](https://raw.githubusercontent.com/NickManos/Screenshots/master/map.png "Basic Google Map") 

So we have a map but now need to display our users on it. First, we need to make sure our users have latitude and longitude attributes in the database stored as floats with a migration like this:

```ruby
      class AddLatitudeAndLongitudeToUsers < ActiveRecord::Migration
        def change
          add_column :users, :latitude, :float
          add_column :users, :longitude, :float
        end
      end
```

Now we need to get Geocoder working to convert their address fields into a latitude and longitude. This is actually very simple. Just add this to your user model.

```ruby
      geocoded_by :full_street_address
      after_validation :geocode
```

And viola! Geocoder will automatically take the address you feed it and fill in the latitude and longitude of whatever model you attach it to. Last thing to do is build the full_street_address that you give Geocoder above. We'll do this with a bit of Ruby, also in the user model.
      
```ruby
      def full_street_address
          full = address1 + ' ' + address2 + ' ' + city + ' ' + state + ' ' + zip
          return full_address if full.blank?
          full
      end
```

That's it! Note that the full_address that's returned above if the address you build is blank was a default address value we added to users just in case a blank address slipped thorough. It turned out that trying to place markers that had no address data broke the map.

So we have a map and we have users with latitude and longitude data. The next step is to build the markers from that data so that the addMarkers function in our JavaScript has something to work with. To do this we'll go to the controller for whatever page you rendered the map on. My map was in the index.html of a view called Map so in the index action of my Map controller I add this code:

```ruby
      def index
          @users = User.all
          @hash = Gmaps4rails.build_markers(@users) do |user, marker|
                marker.lat user.latitude
                marker.lng user.longitude
                marker.json(id: user.id)
                marker.infowindow render_to_string(partial: '/users/popup_template',
                                         locals: { object: user })
           end
      end
```

This will take our users and set the marker's lat/lng attributes from the latitude/longitude of our users. The content of the popup window that appears when you click the marker can be modified in the app/view/users/popup_template partial.

That's everything! You should now have a working map with your users displayed as points that can be moused over. From here you could add custom icons for each user to the map or have some of their info display when their icon is clicked as can be seen in the screenshot below. The documentation on the Google Maps for Rails gem has information on other things you can accomplish with the map. Thanks for reading!

![map with instrument markers](https://raw.githubusercontent.com/NickManos/Screenshots/master/map_with_markers2.png "Map with instrument markers and user info.") 

