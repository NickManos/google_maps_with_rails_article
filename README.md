## Google Maps with Ruby on Rails

Google Maps is a amazing api that can give an app a ton of added functionality with a very small ammout of code.

In this article I will show you what is necessary to get it running with Ruby on Rails and display your users on a map.

We will use 2 gems to accomplish this:

      Google maps for Rails-
      https://github.com/apneadiving/Google-Maps-for-Rails
      
      Geocoder-
      https://github.com/alexreisner/geocoder
            
  So how do Rails, the map, the geocoder, and your database fit together? Google maps for rails will allow us to embed a map in a rails view with a bit of embedded javascript. The geocoder in our users model will take an address which we will build from the various address fields a user fills out when they sign up. It will then give us a latitude and longitude which will be stored in their own columns in our users table. We will use those values to make markers which will represent each user on the map!

First off we need to install the gems listed above. For this simply add to your gemfile:

      gem "gmaps4rails"
      gem "geocoder"

Then remember to bundle install.

Next we need to get the map up and running. For this we will pick a view to hold the map and place the following JavaScript in the file. One thing to note is I was unable to get this working when I linked the JS from a seperate file. It has to be embedded in the view for this to work.

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
      
There are more option the the Google Maps for Rails readme if you need them but this will do for a basic map. Don't worry about the markers for now. We'll get to those in a second. 
