Maps Fragment
=============

We need another Fragment. This time for our Map! You should be getting good at this by now.

### Add a New Fragment

1. Find the package folder.
2. Right-click > New > Fragment > Fragment (Blank)
3. Configure Component:
    - Fragment Name: AccelFragment
    - &#9745; Create layout XML?
    - Layout Name: fragment_accel
    - &#9744; Include fragment factory methods?
    - &#9744; Include interface callbacks?
4. Click **Finish**

*Note: if you need help, look over the steps in Task 1 - Creating a New Fragment or ask for help*

### Add the Fragment to the `SectionsPagerAdapter`

1. Open `MainActivity`
2. Find `SectionsPagerAdapter`. It should be near the bottom of the file.
3. Add our new `AccelFragment` after `HomeFragment`

```java
Fragment[] fragments = { 
    new HomeFragment(),
    new AccelFragment(),  // remember the commas!
    new MapFragment()
};
String[] tabNames = {
    "Home Tab",
    "Accelerometer", // remember the commas!
    "Map"
};
```

It's a good time to Run your app again to see that we've created the new Fragment and tab properly. If it's all working well, you'll have 3 tabs: "**HOME TAB**", "**ACCELEROMETER**", and (our new addition) "**MAP**".

# Adding a Map container to the Layout `fragment_map.xml`

We aren't going to put the map directly into the layout. We are going to create a container for the map to go in, and a small area to put some text and buttons.

1) Open fragment_map.xml  
2) Select the **Design** tab at the bottom of the page.  
3) Delete the existing TextView
4) Create a new `LinearLayout (vertical)` by dragging it from the Palette down onto the `FrameLayout` in the Component Tree
5) Create another `LinearLayout (vertical)` inside the above one. 
    - In the Properties on the right, give it the ID: `mapContainer`
6) Create another `LinearLayout (vertical)` under (but not inside) `mapContainer`.  
   [![](images/fragment_map_component_tree.png)](images/fragment_map_component_tree.png)  

7) Switch over to the `Text` tab instead of the `Design` tab.  
   [![](images/fragment_map_text.png)](images/fragment_map_text.png)  

8) We want the Map and the empty space to have a 70-30 ratio, so we are going to add  
   - to mapContainer: `android:layout_weight="70"`
   - to the other LinearLayout: `android:layout_weight="30"`

   The image in step 7 has the lines we want to add underlined, for reference.
   
That's enough layout stuff for now. Let's update our code to load a Map!

# Importing Map Support

Support for creating an using maps doesn't come with the basic-level Android development software. We need to add those files as `dependencies` which Android Studio will then take care of, for us.

Open `build.gradle (Module:app)` and update the dependencies section to look like this.

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support:design:25.3.1'
    compile 'com.android.support:support-v4:25.3.1'

    compile 'com.google.android.gms:play-services-maps:11.0.4'
    compile 'com.google.android.gms:play-services-location:11.0.4'
    compile 'com.google.maps.android:android-maps-utils:0.5'

    testCompile 'junit:junit:4.12'
}
```
*Just add the three files that have `com.google.`, the rest is for reference.*

then we need to Sync our project by hitting the **Sync Now** link in the yellow banner.  
[![](images/gradle_sync.png)](images/gradle_sync.png)  
*If this causes errors, we'll need to fix your SDK. Ask for assistance!*

# Getting Map Permission

When an app installs, it tells the user what kind of things it might do that the user might want to restrict. Since GPS data can be sensitive (you don't want every app to know where you are all the time), we need to specify that we need GPS and Location permissions. Also, because the maps are provided by Google Maps, we need to use a special key that allows us to access the map data.

You can get a key of your own from the [Google API Console](https://developers.google.com/maps/documentation/android-api/signup), but for now we can use one I created earlier.

1) Open `AndroidManifest.xml`
2) Above the `</application>` tag, add the following key definition:
   ```xml
   <meta-data
       android:name="com.google.android.geo.API_KEY"
       android:value="AIzaSyDcEUpJw35tBdnJyKJskWKRi9MMtbR1JG0" />
   ```

3) Below the `</application>` tag, add the following permission declaration:
   ```xml
   <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
   ```

The bottom of your file should look like this:  
[![](images/manifest_add_key_location.png)](images/manifest_add_key_location.png)  

# Creating a Map for the layout

Similar to `AccelFragment`, we need to set up the layout field variables and set them in the `onViewCreated` function. Previously, we connected to the Service Manager, this time, however, we will connect to the Location Manager. But first let's get a map to display.

1) Open `MapFragment`

2) Generate the `onViewCreated` function: 
   1. Right click in the class > Generate > Override
   2. In the Fragment list, select `onViewCreated`
   3. Press **OK**

4) Add in the following snippet to create our Map. It's a bit tricky because it's loading a Fragment inside a Fragment, but we only have to write it once and it will work.

   ```java
   @Override
   public void onViewCreated(View view, Bundle savedInstanceState) {
       super.onViewCreated(view, savedInstanceState);

       // Search for a Fragment with the "tag" mapFragment
       SupportMapFragment supportMapFragment = (SupportMapFragment) getChildFragmentManager().findFragmentByTag("mapFragment");
       // if it doesn't exist...
       if (supportMapFragment == null) {
           // Create the fragment here
           supportMapFragment = new SupportMapFragment();
           // We need to tell the Fragment Manager to load the fragment using this transaction
           // because it's a Fragment inside of a Fragment. If you don't do it this way,
           // you will get errors when swapping tabs.
           getChildFragmentManager()
                   .beginTransaction()
                   .add(R.id.mapContainer, supportMapFragment, "mapFragment")
                   .commitNow();
       }
    }
   ```

   *`R.id.mapContainer`, etc, is generated from our fragment_map.xml file*

Run your app again and when you swipe to the **MAP** tab, you'll be able to see the map!

# My Location

### Layout

Add a couple of TextViews to `fragment_map.xml` and call them 
- textLongitude
- textLatitude

[![](images/layout_text_latlng.png)](images/layout_text_latlng.png)  

### Code

Open the file `MapFragment`

Similar to the `SensorEventListener` with the Accelerometer, we need to create a `LocationListener` and another class called `OnMapReadyCallback` which we use to put shapes and markers on the map.

At the bottom of `MapFragment`, create a `OnMapReadyCallback` using the following code. All it does at the moment, is check if we have the correct permission then it lets the map know we want to track our location.

```java
class MyMapReadyCallback implements OnMapReadyCallback {
    @Override
    public void onMapReady(GoogleMap googleMap) {
        // Check if we have the correct permissions and throw an exception if we don't
        if (ContextCompat.checkSelfPermission(getContext(), Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            throw new SecurityException("Requires Access Fine Location Permission");
        }

        // Enable the "My Location" feature
        googleMap.setMyLocationEnabled(true);
    }
}
```

We need to add our callback to our `supportMapFragment` that we created in `onViewCreated`. Add this to the bottom of `onViewCreated`:

```java
// When the map is ready, this runs the function `onMapReady` in `MyMapReadyCallback`
supportMapFragment.getMapAsync(new MyMapReadyCallback());
```

You can run the app now and see that there's now a "My Location" button in the top right corner of the map, and when you click it, your location will show up!


This isn't enough to really use our location though. We need to set up a `LocationListener` and push the values to the two `TextView`s we created earlier. Let's set up the text views:

At the top of the class `MapFragment` add these two `TextView`s below the class declaration:

```java
public class MapFragment extends Fragment {
    TextView textLongitude;
    TextView textLatitude;
```

Now we'll create our `LocationListener`. At the bottom of the file (still inside the class), add this code:

```java
class MyLocationListener implements LocationListener {
    @Override
    // This function will get called every time out location updates.
    public void onLocationChanged(Location location) {
        // Display our Latitude
        textLatitude.setText("Latitude: " + location.getLatitude());
        // Display our Longitude
        textLongitude.setText("Longitude: " + location.getLongitude());
    }

    // We're not going to use these ourselves, but we need them here.
    @Override public void onStatusChanged(String s, int i, Bundle bundle) {}
    @Override public void onProviderEnabled(String s) {}
    @Override public void onProviderDisabled(String s) {}
}
```

Now to set these things up, we need to add some more code to `onViewCreated`.  
At the bottom, after `supportMapFragment.getMapAsync(new MyMapReadyCallback());` add:

```java
textLatitude = (TextView) view.findViewById(R.id.textLatitude);
textLongitude = (TextView) view.findViewById(R.id.textLongitude);

// Check if we have the correct permissions and throw an exception if we don't
if (ContextCompat.checkSelfPermission(getContext(), Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
    throw new SecurityException("Requires Access Fine Location Permission");
}
// Get the locationation manager from the system service.
LocationManager locationManager = (LocationManager) getActivity().getSystemService(Context.LOCATION_SERVICE);
String provider = locationManager.getBestProvider(new Criteria(), true);

// Request that we get Location Updates in MyLocationListener
locationManager.requestLocationUpdates(provider, 500, 1, new MyLocationListener());
```

When you run the app now, you'll see the Longitude and Latitude updating in text under the map.

Let's add some features to our map so we can do things like boundaries and markers!

# Map Features

I've created a file called `MyMapOptions` which I've filled with a few shapes as examples. Putting these in a different file means they're a bit easier to keep track of. Let's add this file:

1) Right-click on our package folder `au.edu.catholic.goodshepherd.myapplication`
2) New > Java Class
3) Use the following options
    - Name: MyMapOptions
4) Click **OK**
5) Open the file and paste the following code:

```java
package au.edu.catholic.goodshepherd.myapplication;

import android.graphics.Color;

import com.google.android.gms.maps.model.CircleOptions;
import com.google.android.gms.maps.model.LatLng;
import com.google.android.gms.maps.model.MarkerOptions;
import com.google.android.gms.maps.model.PolygonOptions;
import com.google.android.gms.maps.model.PolylineOptions;

public class MyMapOptions {

    // Oval related map options

    // Coordinates for the centre of the oval
    static LatLng ovalCentre = new LatLng(-20.720620, 139.494870);
    // A circle that fits on the oval
    static CircleOptions ovalCircle = new CircleOptions()
            .radius(35)
            .center(ovalCentre)
            .fillColor(MyColors.transparentRed)
            .strokeWidth(1)
            .strokeColor(MyColors.solidRed);
    // A marker for the oval
    static MarkerOptions ovalMarker = new MarkerOptions()
            .position(ovalCentre)
            .title("Oval");

    // Computer room related map options

    // the computer room coordinates
    static LatLng pcLatLng = new LatLng(-20.721402, 139.494422);

    // a marker for the computer room
    static MarkerOptions pcRoomMarker = new MarkerOptions()
            .position(pcLatLng)
            .title("Computer Room");

    // A solid black line that traces around the sidewalk
    static PolylineOptions schoolSidewalkOutline = new PolylineOptions()
            .color(MyColors.solidBlack)
            .width(3)
            .add(new LatLng(-20.720132, 139.494677))
            .add(new LatLng(-20.720461, 139.495701))
            .add(new LatLng(-20.722229, 139.495055))
            .add(new LatLng(-20.721998, 139.494001))
            .add(new LatLng(-20.720132, 139.494677));

    // An enclosed shape that fills the school area
    static PolygonOptions schoolArea = new PolygonOptions()
            .fillColor(MyColors.transparentBlue)
            .strokeColor(MyColors.solidBlue)
            .strokeWidth(2)
            .add(new LatLng(-20.720205, 139.494721))
            .add(new LatLng(-20.720495, 139.495633))
            .add(new LatLng(-20.722192, 139.495023))
            .add(new LatLng(-20.721941, 139.494096))
            // Put this on the bottom layer
            .zIndex(-1);

    // Some colours that I've predefined for easier reading.
    static class MyColors {
        static int transparentRed =   Color.argb(60, 255,   0,   0);
        static int transparentGreen = Color.argb(60,   0, 255,   0);
        static int transparentBlue =  Color.argb(60,   0,   0, 255);

        static int solidRed =   Color.argb(255, 255,   0,   0);
        static int solidGreen = Color.argb(255,   0, 255,   0);
        static int solidBlue =  Color.argb(255,   0,   0, 255);

        static int solidBlack =  Color.argb(255,   0,   0,  0);
    }
}
```

Back in `MapFragment`, we can add these to the map. In `onMapReady` which is in `MyMapReadyCallback`, after everything we currently have, put in this code:

```java
// Oval Map Parts
googleMap.addCircle(MyMapOptions.ovalCircle);
googleMap.addMarker(MyMapOptions.ovalMarker);

// Computer Room Map Parts
googleMap.addMarker(MyMapOptions.pcRoomMarker);

// School Map Parts
googleMap.addPolygon(MyMapOptions.schoolArea);
googleMap.addPolyline(MyMapOptions.schoolSidewalkOutline);
```

Now when we run app, if you look at your location at the school, you should see:  
[![](images/map_features.png)](images/map_features.png)  

# Comparing our Location to Circles and Polygons

It's good to be able to run some code when we are at a certain place on the map. Having these circles and polygons available to us allows us to know if we are inside or outside a shape.

The best place to check that would be in `MyLocationListener` but it doesn't know about the shapes yet, so for all of the shapes we want to work with back in `MyMapReadyCallback`, we have to save the shapes to field variables and add them to the class.

Near the top of the file, underneath where we added `TextView textLatitude;`, we will create some field variables to track the shapes:

```java
Circle ovalCircle;
Polygon schoolArea;
```

In `onMapReady` in `MyMapReadyCallback`, update the existing code to assigned these variables a value:

```java
// Oval Map Parts
ovalCircle = googleMap.addCircle(MyMapOptions.ovalCircle);
googleMap.addMarker(MyMapOptions.ovalMarker);

// Computer Room Map Parts
googleMap.addMarker(MyMapOptions.pcRoomMarker);

// School Map Parts
schoolArea = googleMap.addPolygon(MyMapOptions.schoolArea);
googleMap.addPolyline(MyMapOptions.schoolSidewalkOutline);
```

Now in `MyLocationListener` we can change `onLocationChanged` to check if we're in the shapes!

```java
public void onLocationChanged(Location location) {
    // Display our Latitude
    textLatitude.setText("Latitude: " + location.getLatitude());
    // Display our Longitude
    textLongitude.setText("Longitude: " + location.getLongitude());

    // We want to use Location.distanceTo which doesn't allow the LatLng types
    // that our shapes use, so we convert it like this.
    Location ovalCentre = new Location("");
    ovalCentre.setLongitude(ovalCircle.getCenter().longitude);
    ovalCentre.setLatitude(ovalCircle.getCenter().latitude);

    if (location.distanceTo(ovalCentre) < ovalCircle.getRadius()) {
        ovalCircle.setFillColor(MyMapOptions.MyColors.transparentGreen);
        ovalCircle.setStrokeColor(MyMapOptions.MyColors.solidGreen);
    } else {
        ovalCircle.setFillColor(MyMapOptions.MyColors.transparentRed);
        ovalCircle.setStrokeColor(MyMapOptions.MyColors.solidRed);
    }

    // We want to use PolyUtil.containsLocation which doesn't allow the Location types
    // that our location uses, so we convert it like this...
    LatLng locationLatLng = new LatLng(location.getLatitude(), location.getLongitude());

    if (PolyUtil.containsLocation(locationLatLng, schoolArea.getPoints(), false)) {
        schoolArea.setStrokeColor(MyMapOptions.MyColors.solidGreen);
    } else {
        schoolArea.setStrokeColor(MyMapOptions.MyColors.solidBlack);
    }
}
```

Now when you're inside the circle or polygon, they will change colour! You could write code to do anything though, display a message, change the layout, and so much more. 

That's all we'll be doing for the map, today. 

## Next: [Task 4 - Cameras and Barcodes](task4_camera.md)